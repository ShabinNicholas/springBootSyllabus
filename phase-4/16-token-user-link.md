# 16. Link Tokens to Users

> Record which user owns each refresh token with a `@ManyToOne` relationship, then test the whole logout flow.

## Why?

Right now a `refresh_token` row doesn't say whose token it is. A cleaner design links it to the user:

```text
User (1) ──────< RefreshToken (many)
                  ├── token
                  ├── revoked
                  └── expiryDate
```

One user can have **many** refresh tokens, for example one from their laptop and one from their phone. Later this makes features like "log out from all devices" possible.

## Step 1 — Add the relationship

In `RefreshToken.java`, add these imports:

```java
import jakarta.persistence.JoinColumn;
import jakarta.persistence.ManyToOne;
```

Add this field below `expiryDate`:

```java
@ManyToOne
@JoinColumn(name = "user_id")
private User user;
```

and its getter and setter:

```java
public User getUser() {
    return user;
}

public void setUser(User user) {
    this.user = user;
}
```

| Annotation | Meaning |
|------------|---------|
| `@ManyToOne` | Many refresh tokens belong to one user |
| `@JoinColumn(name = "user_id")` | Store the user's ID in a column called `user_id` |

?> `User` is in the same package as `RefreshToken` (`entity`), so it needs no import.

## Step 2 — Set the user at login

In `AuthService.login()`, add one line before `refreshTokenRepository.save(...)`:

```java
refreshTokenEntity.setUser(user);
```

So it becomes:

```java
RefreshToken refreshTokenEntity = new RefreshToken();

refreshTokenEntity.setToken(refreshToken);
refreshTokenEntity.setRevoked(false);
refreshTokenEntity.setExpiryDate(
        new Date(System.currentTimeMillis() + 7L * 24 * 60 * 60 * 1000)
);

refreshTokenEntity.setUser(user);

refreshTokenRepository.save(refreshTokenEntity);
```

Restart the app. Hibernate adds a `user_id` column to `refresh_token`.

## Step 3 — Verify the link

Log in as Alice, then run:

```sql
SELECT id, revoked, expiry_date, user_id
FROM refresh_token
ORDER BY id DESC;
```

```text
 id | revoked | expiry_date | user_id
----+---------+-------------+---------
  8 | f       | ...         |       2
```

If Alice's user ID is 2, `user_id = 2` on the newest row confirms the link. Older rows have `user_id` empty (`NULL`) because they were created before this step. That's fine.

```text
users                       refresh_token
id = 2                      id = 8
email = alice@example.com ← user_id = 2
                            revoked = false
```

## Step 4 — Test the whole logout flow

1. **Log in** as Alice and copy the `refreshToken`.
2. **Check the database:** the newest `refresh_token` row has `revoked = false` and Alice's `user_id`.
3. **Log out:** <span class="method post">POST</span> `/auth/logout` with the refresh token → `{"message": "Logout successful"}`.
4. **Check again:** the same row now has `revoked = true`.
5. **Refresh:** <span class="method post">POST</span> `/auth/refresh` with the same token → **`401`** `{"message": "Refresh token has been revoked"}`.
6. **Restart the app** and try step 5 again. Still **`401`**: unlike the in-memory version, the revocation survives a restart. ✅

## 🎉 Phase 4 complete

```text
/auth/signup  → user created, BCrypt password, role = USER
/auth/login   → access token (15 min) + refresh token (7 days, stored with user_id)
/students     → needs a valid ACCESS token; role decides what's allowed
/auth/refresh → valid, stored, un-revoked REFRESH token → new access token
/auth/logout  → refresh token revoked in PostgreSQL
```

| Situation | Result |
|-----------|--------|
| No token on `/students` | `403` |
| Refresh token used as Bearer token | `403` |
| USER calls POST / PUT / DELETE | `403` |
| ADMIN calls POST / PUT / DELETE | `201` / `200` / `204` |
| Access token sent to `/auth/refresh` | `401` |
| Revoked refresh token | `401` |

?> **Ideas for later:** logout from all devices (revoke every token with the user's `user_id`), deleting expired rows from `refresh_token`, a `401` for wrong login details instead of `500`, and `@PreAuthorize` method security.

## ✅ Checkpoint

- [ ] `RefreshToken` has a `@ManyToOne` `user`
- [ ] New refresh tokens are saved with the right `user_id`
- [ ] Logout sets `revoked = true` and refresh then returns `401`, even after a restart

Next: **[Final Code](phase-4/final-code.md)**: every changed file in its finished Phase 4 state →
