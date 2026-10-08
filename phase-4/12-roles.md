# 12. Roles

> Give every user a role, put it in the tokens, and hand it to Spring Security.

Authentication answers *who are you?* Now **authorization**: *what are you allowed to do?*

```text
USER  → can view students
ADMIN → can create, update and delete students
```

## Step 1 — Add a role to User

Open `entity/User.java` and add a field below `password`:

```java
private String role;
```

and its getter and setter:

```java
public String getRole() {
    return role;
}

public void setRole(String role) {
    this.role = role;
}
```

?> We store the role as a `String` to keep things simple. A Java `enum` (`Role.USER`, `Role.ADMIN`) is safer against typos and is a good later improvement.

Restart the app. Hibernate adds a `role` column to the `users` table.

## Step 2 — Every signup gets USER

In `AuthService.signup()`, right after `user.setPassword(hashedPassword);`, add:

```java
user.setRole("USER");
```

So that part reads:

```java
String hashedPassword =
        passwordEncoder.encode(signupRequest.getPassword());

user.setPassword(hashedPassword);
user.setRole("USER");

User savedUser = userRepository.save(user);
```

!> **The backend decides the role, never the client.** If signup read a role from the request, anyone could send `"role": "ADMIN"` and make themselves an admin. `SignupRequest` has no `role` field, so that's impossible.

## Step 3 — Read the role from a token

In `JwtService.java`, add this method below `extractTokenType()`:

```java
public String extractRole(String token) {

    SecretKey key = Keys.hmacShaKeyFor(
            secretKey.getBytes(StandardCharsets.UTF_8)
    );

    return Jwts.parser()
            .verifyWith(key)
            .build()
            .parseSignedClaims(token)
            .getPayload()
            .get("role", String.class);
}
```

## Step 4 — Put the role in the access token

Change `generateAccessToken()` so it takes the role as a **second parameter** and adds it as a claim:

```java
public String generateAccessToken(String email, String role) {

    SecretKey key = Keys.hmacShaKeyFor(
            secretKey.getBytes(StandardCharsets.UTF_8)
    );

    return Jwts.builder()
            .subject(email)
            .claim("type", "access")
            .claim("role", role)
            .issuedAt(new Date())
            .expiration(new Date(
                    System.currentTimeMillis() + 15 * 60 * 1000
            ))
            .signWith(key)
            .compact();
}
```

Changing the method's parameters breaks the two places that call it with only an email. Fix both now:

**In `AuthService.login()`:**

```java
String accessToken =
        jwtService.generateAccessToken(
                user.getEmail(),
                user.getRole()
        );
```

**In `JwtService.refreshAccessToken()`**, replace the last line `return generateAccessToken(email);` with:

```java
String role = extractRole(refreshToken);

return generateAccessToken(email, role);
```

?> **Every new variable needs to come from somewhere.** `role` is a parameter in `generateAccessToken`, and in `refreshAccessToken` we read it from the token with `extractRole()`. If VS Code underlines `role` in red, one of those two lines is missing.

## Step 5 — Put the role in the refresh token

`refreshAccessToken()` now reads the role **from the refresh token**, so the refresh token must contain it too. Change `generateRefreshToken()`:

```java
public String generateRefreshToken(String email, String role) {

    SecretKey key = Keys.hmacShaKeyFor(
            secretKey.getBytes(StandardCharsets.UTF_8)
    );

    return Jwts.builder()
            .subject(email)
            .claim("type", "refresh")
            .claim("role", role)
            .issuedAt(new Date())
            .expiration(new Date(
                    System.currentTimeMillis() + 7L * 24 * 60 * 60 * 1000
            ))
            .signWith(key)
            .compact();
}
```

And update the call in `AuthService.login()`:

```java
String refreshToken =
        jwtService.generateRefreshToken(
                user.getEmail(),
                user.getRole()
        );
```

```text
User logs in → database → email + role
                 ↓                    ↓
          Access Token          Refresh Token
          role = USER           role = USER
          type = access         type = refresh
```

## Step 6 — Give Spring Security the role

In `JwtAuthenticationFilter.java`, inside `if ("access".equals(tokenType))`, replace the block with:

```java
if ("access".equals(tokenType)) {

    String email = jwtService.extractEmail(token);

    String role = jwtService.extractRole(token);

    UsernamePasswordAuthenticationToken authentication =
            new UsernamePasswordAuthenticationToken(
                    email,
                    null,
                    java.util.Collections.singletonList(
                            new org.springframework.security.core.authority.SimpleGrantedAuthority(
                                    "ROLE_" + role
                            )
                    )
            );

    SecurityContextHolder
            .getContext()
            .setAuthentication(authentication);

    System.out.println("Authenticated user: " + email);
}
```

The third argument is now a list with one **authority**.

### Why "ROLE_" + role?

| In the database | Spring Security receives |
|-----------------|--------------------------|
| `USER` | `ROLE_USER` |
| `ADMIN` | `ROLE_ADMIN` |

Spring Security's role rules, such as `.hasRole("ADMIN")`, automatically look for an authority with the `ROLE_` prefix. So we add it here.

```text
Database role = USER → JWT role = USER → filter → ROLE_USER → Spring Security
```

Restart the app. There should be no red lines left.

## Step 7 — Test, and fix users without a role

Log in as Alice and decode the new **access** token. You may find the role is missing or `null`. Check the database:

```sql
SELECT id, name, email, role
FROM users
WHERE email = 'alice@example.com';
```

```text
 id | name  |       email       | role
----+-------+-------------------+------
  2 | Alice | alice@example.com |
```

**Alice's role is `NULL`**, because she signed up **before** we added `user.setRole("USER")`. Adding a column doesn't fill it in for existing rows. Fix it:

```sql
UPDATE users
SET role = 'USER'
WHERE email = 'alice@example.com';
```

?> To fix every older user at once: `UPDATE users SET role = 'USER' WHERE role IS NULL;`

## Step 8 — Get fresh tokens

A JWT is fixed when it's created. Changing the database doesn't change tokens you already have, so **log in again**. Decode the new tokens:

```json
{ "sub": "alice@example.com", "type": "access", "role": "USER", "iat": ..., "exp": ... }
```

```json
{ "sub": "alice@example.com", "type": "refresh", "role": "USER", "iat": ..., "exp": ... }
```

Then <span class="method get">GET</span> `/students` with the new access token: **`200 OK`**.

!> **Roles live inside the token.** If you change a user's role in the database, their existing tokens still carry the **old** role until they expire. The user must log in again (or refresh) to get the new role.

## ✅ Checkpoint

- [ ] `User` has a `role` field; new signups get `USER`
- [ ] Both tokens contain `role`
- [ ] The filter gives Spring Security `ROLE_<role>`
- [ ] Alice's role is `USER` and `GET /students` still returns `200`

Next: **[13. Role Rules](phase-4/13-role-rules.md)** →
