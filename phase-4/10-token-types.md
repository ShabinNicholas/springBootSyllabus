# 10. Token Types

> Mark each token as `access` or `refresh`, and only accept the right one in each place.

We'll add a **claim** (an extra field in the payload) called `type`:

```json
{ "sub": "alice@example.com", "type": "access" }
```

```json
{ "sub": "alice@example.com", "type": "refresh" }
```

Then `/students` accepts only `access`, and `/auth/refresh` accepts only `refresh`.

## Step 1 — Mark the access token

In `JwtService.generateAccessToken()`, add `.claim("type", "access")` before `.issuedAt(...)`:

```java
return Jwts.builder()
        .subject(email)
        .claim("type", "access")
        .issuedAt(new Date())
        .expiration(new Date(System.currentTimeMillis() + 15 * 60 * 1000))
        .signWith(key)
        .compact();
```

## Step 2 — Mark the refresh token

In `generateRefreshToken()`, add `.claim("type", "refresh")`:

```java
public String generateRefreshToken(String email) {

    SecretKey key = Keys.hmacShaKeyFor(
            secretKey.getBytes(StandardCharsets.UTF_8)
    );

    return Jwts.builder()
            .subject(email)
            .claim("type", "refresh")
            .issuedAt(new Date())
            .expiration(new Date(System.currentTimeMillis() + 7L * 24 * 60 * 60 * 1000))
            .signWith(key)
            .compact();
}
```

## Step 3 — Read the type

Add this method below `extractEmail()`:

```java
public String extractTokenType(String token) {

    SecretKey key = Keys.hmacShaKeyFor(
            secretKey.getBytes(StandardCharsets.UTF_8)
    );

    return Jwts.parser()
            .verifyWith(key)
            .build()
            .parseSignedClaims(token)
            .getPayload()
            .get("type", String.class);
}
```

It returns `"access"` or `"refresh"` (or `null` for an old token made before we added the claim).

## Step 4 — /students accepts only access tokens

In `JwtAuthenticationFilter.java`, replace:

```java
if (jwtService.validateToken(token)) {

    String email =
            jwtService.extractEmail(token);

    UsernamePasswordAuthenticationToken authentication =
            new UsernamePasswordAuthenticationToken(
                    email,
                    null,
                    java.util.Collections.emptyList()
            );

    SecurityContextHolder
            .getContext()
            .setAuthentication(authentication);

    System.out.println("Authenticated user: " + email);
}
```

with:

```java
if (jwtService.validateToken(token)) {

    String tokenType =
            jwtService.extractTokenType(token);

    if ("access".equals(tokenType)) {

        String email =
                jwtService.extractEmail(token);

        UsernamePasswordAuthenticationToken authentication =
                new UsernamePasswordAuthenticationToken(
                        email,
                        null,
                        java.util.Collections.emptyList()
                );

        SecurityContextHolder
                .getContext()
                .setAuthentication(authentication);

        System.out.println("Authenticated user: " + email);
    }
}
```

```text
Valid JWT → check type → type = access?
                           YES → authenticate
                           NO  → don't authenticate
```

?> **Why `"access".equals(tokenType)` and not `tokenType.equals("access")`?** If `tokenType` is `null`, the second form crashes with a `NullPointerException`. Putting the fixed string first is null-safe.

Restart the app.

## Step 5 — Test: access token still works

Log in for **fresh** tokens (old ones have no `type` claim). Send <span class="method get">GET</span> `/students` with the **access** token: **`200 OK`**.

## Step 6 — Test: refresh token is rejected by /students

Send the same request but paste the **refresh** token as the Bearer token: **`403 Forbidden`**.

```text
Refresh token → valid JWT ✅ → type = refresh → not authenticated ❌ → /students → 403
```

A valid JWT isn't enough anymore: it must also be the right **type**.

| Token | Lifetime | Can access `/students` |
|-------|----------|------------------------|
| Access token | 15 minutes | ✅ Yes |
| Refresh token | 7 days | ❌ No |

## Step 7 — /auth/refresh accepts only refresh tokens

In `JwtService`, replace `refreshAccessToken()` with:

```java
public String refreshAccessToken(String refreshToken) {

    if (!validateToken(refreshToken)) {
        throw new RuntimeException("Invalid or expired refresh token");
    }

    String tokenType = extractTokenType(refreshToken);

    if (!"refresh".equals(tokenType)) {
        throw new RuntimeException("Invalid refresh token");
    }

    String email = extractEmail(refreshToken);

    return generateAccessToken(email);
}
```

Restart the app.

## Step 8 — Test both cases

**Correct:** <span class="method post">POST</span> `/auth/refresh` with your **refresh** token → **`200 OK`** with a new access token.

**Wrong:** put your **access** token in the body:

```json
{
  "refreshToken": "YOUR_ACCESS_TOKEN"
}
```

It's rejected ✅, but with **`500 Internal Server Error`**, because we throw a plain `RuntimeException`. A bad token is the client's mistake, not a server failure, so it should be `401 Unauthorized`. That's the next page.

## ✅ Checkpoint

- [ ] Access tokens have `type = access`, refresh tokens have `type = refresh`
- [ ] Refresh token on `/students` → `403`
- [ ] Access token on `/auth/refresh` → rejected (currently `500`)

Next: **[11. Invalid Token Errors](phase-4/11-invalid-token.md)** →
