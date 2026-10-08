# 2. Access Token

> Generate a JWT access token and return it from `/auth/login`.

## Step 1 — Generate the access token

Replace `JwtService.java` with:

```java
package com.example.student_api.service;

import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.security.Keys;
import org.springframework.stereotype.Service;

import javax.crypto.SecretKey;
import java.nio.charset.StandardCharsets;
import java.util.Date;

@Service
public class JwtService {

    private final String secretKey =
            "my-super-secret-key-for-student-api-123456789";

    public String generateAccessToken(String email) {

        SecretKey key = Keys.hmacShaKeyFor(
                secretKey.getBytes(StandardCharsets.UTF_8)
        );

        return Jwts.builder()
                .subject(email)
                .issuedAt(new Date())
                .expiration(new Date(System.currentTimeMillis() + 15 * 60 * 1000))
                .signWith(key)
                .compact();
    }
}
```

| Code | Meaning |
|------|---------|
| `Keys.hmacShaKeyFor(...)` | Turns our secret string into a signing key |
| `.subject(email)` | Puts the user's email in the token (the "subject") |
| `.issuedAt(new Date())` | When the token was created |
| `.expiration(...)` | When it stops working: now + 15 minutes |
| `.signWith(key)` | Signs it, so we can later check nobody changed it |
| `.compact()` | Builds the final `xxxxx.yyyyy.zzzzz` string |

?> `15 * 60 * 1000` is 15 minutes in milliseconds (15 × 60 seconds × 1000 ms).

!> **The secret must be at least 32 characters** (256 bits). With a shorter one, `Keys.hmacShaKeyFor` throws a `WeakKeyException`. Our example secret is 45 characters.

Restart the app.

## Step 2 — Return a token from login

Open `AuthService.java`.

**1. Add a field:**

```java
private final JwtService jwtService;
```

**2. Update the constructor:**

```java
public AuthService(
        UserRepository userRepository,
        PasswordEncoder passwordEncoder,
        JwtService jwtService) {

    this.userRepository = userRepository;
    this.passwordEncoder = passwordEncoder;
    this.jwtService = jwtService;
}
```

?> `JwtService` is in the same package as `AuthService` (`service`), so no import is needed.

**3. Replace `login()`:**

```java
public String login(LoginRequest loginRequest) {

    User user = userRepository.findByEmail(loginRequest.getEmail())
            .orElseThrow(() -> new RuntimeException("Invalid email or password"));

    boolean passwordMatches = passwordEncoder.matches(
            loginRequest.getPassword(),
            user.getPassword()
    );

    if (!passwordMatches) {
        throw new RuntimeException("Invalid email or password");
    }

    return jwtService.generateAccessToken(user.getEmail());
}
```

```text
Login → check email → check password → correct? → generate access token → return it
```

**4. Update `AuthController`:** change the return type from `boolean` to `String`:

```java
@PostMapping("/login")
public String login(@RequestBody LoginRequest loginRequest) {
    return authService.login(loginRequest);
}
```

Restart the app.

## Step 3 — Get your first access token 🔐

In Bruno, send <span class="method post">POST</span> `http://localhost:8080/auth/login`:

```json
{
  "email": "alice@example.com",
  "password": "123456"
}
```

You get a long string like:

```text
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJhbGljZUBleGFtcGxlLmNvbSIsImlhdCI6MTc5MTE5MTExMCwiZXhwIjoxNzkxMTkyMDEwfQ.twQMvkD0fZgZlwqRQNhHfaEDhzwydCG8TwanaUCOZt4
```

That's your **access token**. Yours will be different, and it changes on every login because the issued-at time changes.

It has three parts separated by dots:

```text
eyJhbGciOiJIUzI1NiJ9                        ← HEADER    {"alg":"HS256"}
.
eyJzdWIiOiJhbGljZUBleGFtcGxlLmNvbSIs...     ← PAYLOAD   {"sub":"alice@example.com","iat":...,"exp":...}
.
twQMvkD0fZgZlwqRQNhHfaEDhzwydCG8TwanaUCOZt4 ← SIGNATURE
```

?> A wrong password now returns `500 Internal Server Error`, because we throw a plain `RuntimeException`. We clean up error responses later in this phase.

## ✅ Checkpoint

- [ ] `JwtService.generateAccessToken()` exists
- [ ] `/auth/login` returns a JWT string

Next: **[3. Login Response](phase-4/03-login-response.md)** →
