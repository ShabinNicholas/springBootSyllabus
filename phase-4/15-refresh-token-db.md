# 15. Refresh Tokens in PostgreSQL

> Save every refresh token in the database, and make the database decide whether it's revoked.

```text
Before:  refresh token → Java Set (lost on restart)
After:   refresh token → PostgreSQL (refresh_token table) → revoked = true / false
```

## Step 1 — Create the RefreshToken entity

Create `src/main/java/com/example/student_api/entity/RefreshToken.java`:

```java
package com.example.student_api.entity;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;

import java.util.Date;

@Entity
public class RefreshToken {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(length = 1000)
    private String token;

    private boolean revoked;

    private Date expiryDate;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getToken() {
        return token;
    }

    public void setToken(String token) {
        this.token = token;
    }

    public boolean isRevoked() {
        return revoked;
    }

    public void setRevoked(boolean revoked) {
        this.revoked = revoked;
    }

    public Date getExpiryDate() {
        return expiryDate;
    }

    public void setExpiryDate(Date expiryDate) {
        this.expiryDate = expiryDate;
    }
}
```

Hibernate creates a `refresh_token` table with `id`, `token`, `revoked` and `expiry_date`.

?> **Why `@Column(length = 1000)`?** By default a `String` column holds up to **255** characters. A refresh token is roughly 200 characters, and longer with a long email address. Without the bigger length, saving a long token fails with `value too long for type character varying(255)`. If your table already exists with 255, run `ALTER TABLE refresh_token ALTER COLUMN token TYPE varchar(1000);`.

?> `boolean` getters are named `isRevoked()`, not `getRevoked()`. That's the Java convention for booleans.

## Step 2 — Create RefreshTokenRepository

Create `src/main/java/com/example/student_api/repository/RefreshTokenRepository.java`:

```java
package com.example.student_api.repository;

import com.example.student_api.entity.RefreshToken;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.Optional;

public interface RefreshTokenRepository
        extends JpaRepository<RefreshToken, Long> {

    Optional<RefreshToken> findByToken(String token);
}
```

`findByToken` works like `findByEmail` in Phase 3: Spring Data JPA builds the query from the method name.

```text
Refresh token → findByToken() → matching row → revoked true / false
```

## Step 3 — Give AuthService the repository

In `AuthService.java`, add:

```java
import com.example.student_api.entity.RefreshToken;
import com.example.student_api.repository.RefreshTokenRepository;
```

Add a field:

```java
private final RefreshTokenRepository refreshTokenRepository;
```

and update the constructor:

```java
public AuthService(
        UserRepository userRepository,
        PasswordEncoder passwordEncoder,
        JwtService jwtService,
        RefreshTokenRepository refreshTokenRepository) {

    this.userRepository = userRepository;
    this.passwordEncoder = passwordEncoder;
    this.jwtService = jwtService;
    this.refreshTokenRepository = refreshTokenRepository;
}
```

## Step 4 — Save the refresh token at login

In `AuthService.java`, add:

```java
import java.util.Date;
```

In `login()`, right **after** the `generateRefreshToken(...)` call, add:

```java
RefreshToken refreshTokenEntity = new RefreshToken();

refreshTokenEntity.setToken(refreshToken);
refreshTokenEntity.setRevoked(false);
refreshTokenEntity.setExpiryDate(
        new Date(System.currentTimeMillis() + 7L * 24 * 60 * 60 * 1000)
);

refreshTokenRepository.save(refreshTokenEntity);
```

So the end of `login()` is:

```java
String refreshToken =
        jwtService.generateRefreshToken(
                user.getEmail(),
                user.getRole()
        );

RefreshToken refreshTokenEntity = new RefreshToken();

refreshTokenEntity.setToken(refreshToken);
refreshTokenEntity.setRevoked(false);
refreshTokenEntity.setExpiryDate(
        new Date(System.currentTimeMillis() + 7L * 24 * 60 * 60 * 1000)
);

refreshTokenRepository.save(refreshTokenEntity);

LoginResponse response = new LoginResponse();

response.setAccessToken(accessToken);
response.setRefreshToken(refreshToken);

return response;
```

```text
Login → generate refresh token → RefreshToken entity (revoked = false) → save to PostgreSQL
```

## Step 5 — Give JwtService the repository

In `JwtService.java`, add:

```java
import com.example.student_api.entity.RefreshToken;
import com.example.student_api.repository.RefreshTokenRepository;
```

Add a field below the `Set`:

```java
private final RefreshTokenRepository refreshTokenRepository;
```

and a constructor:

```java
public JwtService(RefreshTokenRepository refreshTokenRepository) {
    this.refreshTokenRepository = refreshTokenRepository;
}
```

?> `secretKey` is still filled by `@Value`. A class can use constructor injection for some fields and `@Value` for others.

Don't remove the `Set` yet. We switch over gradually.

## Step 6 — Check the database instead of the Set

At the beginning of `refreshAccessToken()`, replace:

```java
if (revokedRefreshTokens.contains(refreshToken)) {
    throw new InvalidTokenException("Refresh token has been revoked");
}
```

with:

```java
RefreshToken refreshTokenEntity =
        refreshTokenRepository.findByToken(refreshToken)
                .orElseThrow(() ->
                        new InvalidTokenException("Invalid refresh token"));

if (refreshTokenEntity.isRevoked()) {
    throw new InvalidTokenException("Refresh token has been revoked");
}
```

Now a refresh token must **exist in the database** and **not be revoked**:

```text
Refresh token → PostgreSQL → found? → revoked?
                               NO → 401      YES → 401
```

!> **Old refresh tokens stop working.** Tokens issued before Step 4 aren't in the database, so `/auth/refresh` now answers `401 "Invalid refresh token"` for them. Log in again to get a stored one.

## Step 7 — Revoke in the database

Delete the `Set` field:

```java
private final Set<String> revokedRefreshTokens =
        ConcurrentHashMap.newKeySet();
```

Replace `revokeRefreshToken()` with:

```java
public void revokeRefreshToken(String refreshToken) {

    RefreshToken refreshTokenEntity =
            refreshTokenRepository.findByToken(refreshToken)
                    .orElseThrow(() ->
                            new InvalidTokenException("Invalid refresh token"));

    refreshTokenEntity.setRevoked(true);

    refreshTokenRepository.save(refreshTokenEntity);
}
```

And remove the two imports you no longer need:

```java
import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;
```

```text
LOGIN   → refresh token saved, revoked = false
LOGOUT  → find token in PostgreSQL → revoked = true → save
REFRESH → find token in PostgreSQL → revoked? YES → 401 ❌  NO → new access token ✅
```

The **database is now the source of truth** for whether a refresh token is revoked.

## Step 8 — Check nothing old is left

In `JwtService.java`:

- There is **no** `revokedRefreshTokens` anywhere.
- There is **no** `import java.util.Set;` or `import java.util.concurrent.ConcurrentHashMap;`.
- There **is** `private final RefreshTokenRepository refreshTokenRepository;` and its constructor.

## Step 9 — Check the table

Start the app (`.\mvnw spring-boot:run`) and in psql or pgAdmin run:

```sql
SELECT * FROM refresh_token;
```

The table exists. It may have 0 rows until someone logs in.

## ✅ Checkpoint

- [ ] `entity/RefreshToken.java` and `repository/RefreshTokenRepository.java` exist
- [ ] Login saves a row in `refresh_token`
- [ ] `refreshAccessToken()` and `revokeRefreshToken()` use the database
- [ ] No in-memory `Set` is left

Next: **[16. Link Tokens to Users](phase-4/16-token-user-link.md)** →
