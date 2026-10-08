# 9. Refresh Endpoint

> Build `POST /auth/refresh` to swap a refresh token for a new access token.

The access token expires after 15 minutes. Instead of logging in again, the client sends its refresh token:

<span class="method post">POST</span> `/auth/refresh`

```json
{ "refreshToken": "your-refresh-token" }
```

and gets back a new access token.

## Step 1 — Create RefreshTokenRequest

Create `src/main/java/com/example/student_api/dto/RefreshTokenRequest.java`:

```java
package com.example.student_api.dto;

public class RefreshTokenRequest {

    private String refreshToken;

    public String getRefreshToken() {
        return refreshToken;
    }

    public void setRefreshToken(String refreshToken) {
        this.refreshToken = refreshToken;
    }
}
```

## Step 2 — Add refreshAccessToken() to JwtService

Add this method below `extractEmail()`:

```java
public String refreshAccessToken(String refreshToken) {

    if (!validateToken(refreshToken)) {
        throw new RuntimeException("Invalid or expired refresh token");
    }

    String email = extractEmail(refreshToken);

    return generateAccessToken(email);
}
```

```text
Refresh token → validateToken() → valid? → extractEmail() → generateAccessToken() → new access token
```

## Step 3 — Add the endpoint to AuthController

The controller needs `JwtService`. In `AuthController.java`, add these imports:

```java
import com.example.student_api.dto.RefreshTokenRequest;
import com.example.student_api.service.JwtService;
```

Add a field:

```java
private final JwtService jwtService;
```

Change the constructor to:

```java
public AuthController(
        AuthService authService,
        JwtService jwtService) {

    this.authService = authService;
    this.jwtService = jwtService;
}
```

Then add this method below `login()`:

```java
@PostMapping("/refresh")
public LoginResponse refreshToken(
        @RequestBody RefreshTokenRequest request) {

    String newAccessToken =
            jwtService.refreshAccessToken(request.getRefreshToken());

    LoginResponse response = new LoginResponse();

    response.setAccessToken(newAccessToken);

    return response;
}
```

`/auth/refresh` falls under `anyRequest().permitAll()`, so it doesn't need an `Authorization` header.

Restart the app.

## Step 4 — Test

1. Log in and copy the **refreshToken**.
2. Send <span class="method post">POST</span> `http://localhost:8080/auth/refresh` with **No Auth** and:

```json
{
  "refreshToken": "your-refresh-token-here"
}
```

You get:

```json
{
  "accessToken": "new-access-token",
  "refreshToken": null
}
```

The new access token works. `refreshToken` is `null` because we only set the access token in the response.

## Step 5 — Return the refresh token too

In the `refreshToken()` controller method, change:

```java
response.setAccessToken(newAccessToken);
```

to:

```java
response.setAccessToken(newAccessToken);
response.setRefreshToken(request.getRefreshToken());
```

So the method is:

```java
@PostMapping("/refresh")
public LoginResponse refreshToken(
        @RequestBody RefreshTokenRequest request) {

    String newAccessToken =
            jwtService.refreshAccessToken(request.getRefreshToken());

    LoginResponse response = new LoginResponse();

    response.setAccessToken(newAccessToken);
    response.setRefreshToken(request.getRefreshToken());

    return response;
}
```

Restart and send the same request. Now:

```json
{
  "accessToken": "new-access-token",
  "refreshToken": "your-refresh-token"
}
```

?> We send back the **same** refresh token. Some systems issue a brand-new refresh token on every refresh ("refresh token rotation"). That's a more advanced pattern we don't need here.

## Step 6 — Use the new access token

Copy the **new** `accessToken` and send <span class="method get">GET</span> `/students` with it as a Bearer token: **`200 OK`**.

```text
Login → access + refresh token
Access token expires
Refresh token → /auth/refresh → new access token
/students → 200 OK
```

## One problem

Both tokens are JWTs signed with the same secret, and `validateToken()` only checks "is it a valid, unexpired JWT?". So right now:

- a **refresh token** would be accepted by `/students`, and
- an **access token** would be accepted by `/auth/refresh`.

We fix that next.

## ✅ Checkpoint

- [ ] `dto/RefreshTokenRequest.java` exists
- [ ] `/auth/refresh` returns a new access token plus the refresh token
- [ ] The new access token works on `GET /students`

Next: **[10. Token Types](phase-4/10-token-types.md)** →
