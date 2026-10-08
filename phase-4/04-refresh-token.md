# 4. Refresh Token

> Generate a second, longer-lived token at login.

## Why two tokens?

| | Access token | Refresh token |
|---|---|---|
| Lifetime | **15 minutes** | **7 days** |
| Used for | Calling the API (`/students`) | Getting a new access token |
| If stolen | Useful for 15 minutes at most | Can be revoked at logout (end of this phase) |

A short access token limits the damage if it leaks. The refresh token means the user doesn't have to type their password every 15 minutes.

## Step 1 — Add refreshToken to LoginResponse

Replace `dto/LoginResponse.java` with:

```java
package com.example.student_api.dto;

public class LoginResponse {

    private String accessToken;
    private String refreshToken;

    public String getAccessToken() {
        return accessToken;
    }

    public void setAccessToken(String accessToken) {
        this.accessToken = accessToken;
    }

    public String getRefreshToken() {
        return refreshToken;
    }

    public void setRefreshToken(String refreshToken) {
        this.refreshToken = refreshToken;
    }
}
```

## Step 2 — Add generateRefreshToken()

In `JwtService.java`, add this method below `generateAccessToken()`:

```java
public String generateRefreshToken(String email) {

    SecretKey key = Keys.hmacShaKeyFor(
            secretKey.getBytes(StandardCharsets.UTF_8)
    );

    return Jwts.builder()
            .subject(email)
            .issuedAt(new Date())
            .expiration(new Date(System.currentTimeMillis() + 7L * 24 * 60 * 60 * 1000))
            .signWith(key)
            .compact();
}
```

It's the same as the access token, except it expires after **7 days**.

?> **Why `7L`?** 7 days in milliseconds is 604,800,000. That's still small enough for an `int`, but adding the `L` makes the whole calculation use `long`, so it can't overflow if you later make the lifetime longer. It's a good habit with millisecond maths.

Restart the app.

## Step 3 — Generate both tokens at login

In `AuthService.login()`, replace the end of the method:

```java
String accessToken =
        jwtService.generateAccessToken(user.getEmail());

LoginResponse response = new LoginResponse();

response.setAccessToken(accessToken);

return response;
```

with:

```java
String accessToken =
        jwtService.generateAccessToken(user.getEmail());

String refreshToken =
        jwtService.generateRefreshToken(user.getEmail());

LoginResponse response = new LoginResponse();

response.setAccessToken(accessToken);
response.setRefreshToken(refreshToken);

return response;
```

```text
LOGIN → verify email → verify password
                 ↓                ↓
          Access Token      Refresh Token
          15 minutes        7 days
               ↓                ↓
          API requests     Get a new access token
```

Restart the app.

## Step 4 — Test

<span class="method post">POST</span> `http://localhost:8080/auth/login`:

```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiJ9..."
}
```

The two tokens are **different**, because they have different expiry times (and so different signatures).

?> **Look inside a token:** paste a token into a JWT decoder such as [jwt.io](https://jwt.io) to see its payload. Only do this with **test** tokens from your own machine, never with real users' tokens.

We haven't built `/auth/refresh` yet, so the refresh token can't be used yet.

## ✅ Checkpoint

- [ ] `LoginResponse` has `accessToken` and `refreshToken`
- [ ] Login returns two different tokens

Next: **[5. Secret Config](phase-4/05-secret-config.md)** →
