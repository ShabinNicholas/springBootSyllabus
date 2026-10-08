# 14. Logout

> Add `POST /auth/logout`, which revokes the refresh token. First version: in memory.

## What can logout actually do?

A JWT can't be "deleted" from the server once it's issued. The client holds it, and it stays valid until it expires. So logout here means **revoking the refresh token**: after logout, that refresh token can no longer create new access tokens.

```text
Access token  → still works until it expires (at most 15 minutes)
Refresh token → revoked: /auth/refresh refuses it
```

We'll first keep revoked tokens in a Java `Set` (in memory) to see the idea, then move them to PostgreSQL on the next page.

## Step 1 — Create LogoutRequest

Create `src/main/java/com/example/student_api/dto/LogoutRequest.java`:

```java
package com.example.student_api.dto;

public class LogoutRequest {

    private String refreshToken;

    public String getRefreshToken() {
        return refreshToken;
    }

    public void setRefreshToken(String refreshToken) {
        this.refreshToken = refreshToken;
    }
}
```

## Step 2 — A set of revoked tokens

In `JwtService.java`, add these imports:

```java
import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;
```

and add this field **below** the existing `secretKey` field:

```java
private final Set<String> revokedRefreshTokens =
        ConcurrentHashMap.newKeySet();
```

So the top of the class is:

```java
@Service
public class JwtService {

    @Value("${jwt.secret}")
    private String secretKey;

    private final Set<String> revokedRefreshTokens =
            ConcurrentHashMap.newKeySet();

    // existing methods...
}
```

| Code | Meaning |
|------|---------|
| `Set<String>` | A collection of unique strings |
| `revokedRefreshTokens` | The refresh tokens that have been logged out |
| `ConcurrentHashMap.newKeySet()` | A `Set` that's safe when many requests use it at once |

!> **Don't change the `secretKey` line.** It stays `private String secretKey;`. The `Set` is a separate, new field.

## Step 3 — The revoke method

Add to `JwtService`:

```java
public void revokeRefreshToken(String refreshToken) {
    revokedRefreshTokens.add(refreshToken);
}
```

## Step 4 — Refuse revoked tokens

At the **very beginning** of `refreshAccessToken()`, add:

```java
if (revokedRefreshTokens.contains(refreshToken)) {
    throw new InvalidTokenException("Refresh token has been revoked");
}
```

So it starts like this:

```java
public String refreshAccessToken(String refreshToken) {

    if (revokedRefreshTokens.contains(refreshToken)) {
        throw new InvalidTokenException("Refresh token has been revoked");
    }

    if (!validateToken(refreshToken)) {
        throw new InvalidTokenException("Invalid or expired refresh token");
    }

    // existing code...
}
```

Our `InvalidTokenException` handler turns it into `401`.

## Step 5 — The logout endpoint

In `AuthController.java`, add these imports:

```java
import java.util.HashMap;
import java.util.Map;

import com.example.student_api.dto.LogoutRequest;
```

and this method:

```java
@PostMapping("/logout")
public Map<String, String> logout(@RequestBody LogoutRequest request) {

    jwtService.revokeRefreshToken(request.getRefreshToken());

    Map<String, String> response = new HashMap<>();
    response.put("message", "Logout successful");

    return response;
}
```

Restart the app.

## Step 6 — Test the logout flow

1. **Log in** and save the `refreshToken`.
2. **Refresh before logout:** <span class="method post">POST</span> `/auth/refresh` with it → `200` with a new access token.
3. **Log out:** <span class="method post">POST</span> `/auth/logout`:

   ```json
   { "refreshToken": "your-refresh-token" }
   ```

   ```json
   { "message": "Logout successful" }
   ```

4. **Refresh again** with the same token → **`401 Unauthorized`**:

   ```json
   { "message": "Refresh token has been revoked" }
   ```

```text
Login → tokens → refresh works ✅ → logout → token revoked → refresh again → 401 ❌
```

## The problem with memory

The `Set` lives in the application's memory. **Restart the app and it's empty**, so every logged-out refresh token works again. If you ran two copies of the app, each would have its own `Set`. For a real application, revoked tokens must be stored in the **database**. That's the next page.

## ✅ Checkpoint

- [ ] `dto/LogoutRequest.java` exists
- [ ] `POST /auth/logout` returns `"Logout successful"`
- [ ] A logged-out refresh token gets `401`

Next: **[15. Refresh Tokens in PostgreSQL](phase-4/15-refresh-token-db.md)** →
