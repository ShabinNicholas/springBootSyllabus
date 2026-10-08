# 11. Invalid Token Errors

> Create an `InvalidTokenException` and turn it into `401 Unauthorized`.

## Step 1 — Create the exception

Create a new package **`exception`** and inside it `InvalidTokenException.java`:

```java
package com.example.student_api.exception;

public class InvalidTokenException extends RuntimeException {

    public InvalidTokenException(String message) {
        super(message);
    }
}
```

Our own exception class lets the global handler tell "bad token" apart from every other error.

## Step 2 — Throw it from JwtService

In `JwtService.java`, add:

```java
import com.example.student_api.exception.InvalidTokenException;
```

and change `refreshAccessToken()` to use it:

```java
public String refreshAccessToken(String refreshToken) {

    if (!validateToken(refreshToken)) {
        throw new InvalidTokenException("Invalid or expired refresh token");
    }

    String tokenType = extractTokenType(refreshToken);

    if (!"refresh".equals(tokenType)) {
        throw new InvalidTokenException("Invalid refresh token");
    }

    String email = extractEmail(refreshToken);

    return generateAccessToken(email);
}
```

## Step 3 — Handle it globally

In `GlobalExceptionHandler.java`, add:

```java
import com.example.student_api.exception.InvalidTokenException;
```

and this method:

```java
@ExceptionHandler(InvalidTokenException.class)
@ResponseStatus(HttpStatus.UNAUTHORIZED)
public Map<String, String> handleInvalidToken(
        InvalidTokenException ex) {

    Map<String, String> error = new HashMap<>();
    error.put("message", ex.getMessage());

    return error;
}
```

You already have the `HttpStatus`, `ResponseStatus`, `Map` and `HashMap` imports from Phase 2.

Restart the app.

## Step 4 — Test

Send your **access** token to <span class="method post">POST</span> `/auth/refresh`:

```json
{
  "refreshToken": "YOUR_ACCESS_TOKEN"
}
```

Now you get **`401 Unauthorized`**:

```json
{
  "message": "Invalid refresh token"
}
```

```text
Access token  → /students      → ✅ allowed
Refresh token → /students      → ❌ 403
Access token  → /auth/refresh  → ❌ 401 "Invalid refresh token"
Refresh token → /auth/refresh  → ✅ new access token
```

The basic access + refresh token flow is complete. Next: **authorization** with roles.

?> **Still a 500:** login with a wrong email or password throws a plain `RuntimeException`, so it still returns `500`. A good improvement is to throw an exception that maps to `401` with the same message for both cases, like we did here.

## ✅ Checkpoint

- [ ] `exception/InvalidTokenException.java` exists
- [ ] `GlobalExceptionHandler` maps it to `401`
- [ ] Access token on `/auth/refresh` → `401` with `"Invalid refresh token"`

Next: **[12. Roles](phase-4/12-roles.md)** →
