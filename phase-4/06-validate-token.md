# 6. Validate a Token

> Teach `JwtService` to check a token and read the email inside it.

Making tokens is half the job. When a request arrives with a token, the server must answer two questions:

1. **Is this token valid?** (our signature, not expired, not changed)
2. **Who does it belong to?**

## Step 1 — Add validateToken()

In `JwtService.java`, add:

```java
public boolean validateToken(String token) {

    SecretKey key = Keys.hmacShaKeyFor(
            secretKey.getBytes(StandardCharsets.UTF_8)
    );

    try {

        Jwts.parser()
                .verifyWith(key)
                .build()
                .parseSignedClaims(token);

        return true;

    } catch (Exception e) {

        return false;
    }
}
```

`parseSignedClaims()` checks the signature with our key and checks the expiry date. If anything is wrong it throws an exception, and we return `false`.

| Token is... | `validateToken()` |
|-------------|-------------------|
| Signed by us and not expired | `true` |
| Expired | `false` |
| Changed by someone (payload edited) | `false` |
| Signed with a different secret | `false` |
| Not a JWT at all | `false` |

We're not connecting this to Spring Security yet. Restart the app.

## Step 2 — Add extractEmail()

Our token's subject is the user's email. Add:

```java
public String extractEmail(String token) {

    SecretKey key = Keys.hmacShaKeyFor(
            secretKey.getBytes(StandardCharsets.UTF_8)
    );

    return Jwts.parser()
            .verifyWith(key)
            .build()
            .parseSignedClaims(token)
            .getPayload()
            .getSubject();
}
```

```text
JWT → verify signature → read payload → get subject → alice@example.com
```

So later, when a request arrives with `Authorization: Bearer <token>`:

```text
Token → validate → extract email → authenticate the user
```

?> You may notice every method repeats the `Keys.hmacShaKeyFor(...)` line. That's fine for learning. A tidy-up would be a small private method like `getSigningKey()` that they all call.

Restart the app.

## ✅ Checkpoint

- [ ] `JwtService` has `validateToken()` and `extractEmail()`
- [ ] The app starts

Next: **[7. JWT Filter](phase-4/07-jwt-filter.md)** →
