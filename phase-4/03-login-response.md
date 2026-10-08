# 3. Login Response

> Return the token inside a `LoginResponse` DTO instead of a raw string.

## Why?

Right now `/auth/login` returns a bare string. Soon it must return **two** tokens:

```json
{
  "accessToken": "...",
  "refreshToken": "..."
}
```

So we wrap the response in a DTO. We start with the access token only.

## Step 1 — Create LoginResponse

Create `src/main/java/com/example/student_api/dto/LoginResponse.java`:

```java
package com.example.student_api.dto;

public class LoginResponse {

    private String accessToken;

    public String getAccessToken() {
        return accessToken;
    }

    public void setAccessToken(String accessToken) {
        this.accessToken = accessToken;
    }
}
```

## Step 2 — Return it from AuthService

In `AuthService.java`, add this import:

```java
import com.example.student_api.dto.LoginResponse;
```

Replace `login()` with:

```java
public LoginResponse login(LoginRequest loginRequest) {

    User user = userRepository.findByEmail(loginRequest.getEmail())
            .orElseThrow(() -> new RuntimeException("Invalid email or password"));

    boolean passwordMatches = passwordEncoder.matches(
            loginRequest.getPassword(),
            user.getPassword()
    );

    if (!passwordMatches) {
        throw new RuntimeException("Invalid email or password");
    }

    String accessToken =
            jwtService.generateAccessToken(user.getEmail());

    LoginResponse response = new LoginResponse();

    response.setAccessToken(accessToken);

    return response;
}
```

## Step 3 — Update the controller

In `AuthController.java`, add:

```java
import com.example.student_api.dto.LoginResponse;
```

and change the login method:

```java
@PostMapping("/login")
public LoginResponse login(@RequestBody LoginRequest loginRequest) {
    return authService.login(loginRequest);
}
```

Restart the app.

## Step 4 — Test

<span class="method post">POST</span> `http://localhost:8080/auth/login` with Alice's email and password now returns:

```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiJ9..."
}
```

```text
SIGNUP → BCrypt password → Database

LOGIN  → find user → verify BCrypt password → generate JWT → LoginResponse → accessToken
```

## ✅ Checkpoint

- [ ] `dto/LoginResponse.java` exists
- [ ] Login returns `{"accessToken": "..."}`

Next: **[4. Refresh Token](phase-4/04-refresh-token.md)** →
