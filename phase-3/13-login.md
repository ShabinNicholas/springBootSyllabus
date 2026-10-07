# 13. Login

> Check an email and password against the stored BCrypt hash with `POST /auth/login`.

## Step 1 — Create the LoginRequest DTO

Create `src/main/java/com/example/student_api/dto/LoginRequest.java`:

```java
package com.example.student_api.dto;

public class LoginRequest {

    private String email;

    private String password;

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

This is what the user sends to log in:

```json
{
  "email": "alice@example.com",
  "password": "123456"
}
```

## Step 2 — Add login() to AuthService

We're **not** creating tokens yet. First, let's see how Spring checks a password against the BCrypt hash in PostgreSQL.

In `AuthService.java`, add this import:

```java
import com.example.student_api.dto.LoginRequest;
```

and add this method inside the class:

```java
public boolean login(LoginRequest loginRequest) {

    User user = userRepository.findByEmail(loginRequest.getEmail())
            .orElseThrow(() -> new RuntimeException("Invalid email or password"));

    return passwordEncoder.matches(
            loginRequest.getPassword(),
            user.getPassword()
    );
}
```

### What happens here

1. **Find the user by email:**

   ```java
   userRepository.findByEmail(loginRequest.getEmail())
   ```

   ```text
   Email:    alice@example.com
   Password: $2a$10$...
   ```

2. **Compare the passwords:**

   ```java
   passwordEncoder.matches(
           loginRequest.getPassword(),   // what the user typed: 123456
           user.getPassword()            // the stored hash:     $2a$10$...
   );
   ```

   BCrypt hashes what the user typed, using the salt stored inside the hash, and checks whether the result matches. It returns `true` if they match and `false` if they don't.

!> **Argument order matters:** `matches(rawPassword, encodedPassword)`. The plain password goes **first**, the stored hash **second**.

### Why no tokens yet?

We're building authentication one piece at a time:

```text
1. Signup
2. Hash password
3. Store user
4. Login
5. Verify password    ← WE ARE HERE
6. Generate access token
7. Generate refresh token
```

## Step 3 — Add the login endpoint

In `AuthController.java`, add this import:

```java
import com.example.student_api.dto.LoginRequest;
```

and this method:

```java
@PostMapping("/login")
public boolean login(@RequestBody LoginRequest loginRequest) {
    return authService.login(loginRequest);
}
```

The controller now has:

```text
POST /auth/signup
POST /auth/login
```

## Step 4 — Test login

Restart the app. In Bruno:

<span class="method post">POST</span> `http://localhost:8080/auth/login`

**Correct password:**

```json
{
  "email": "alice@example.com",
  "password": "123456"
}
```

Expected:

```json
true
```

**Wrong password:**

```json
{
  "email": "alice@example.com",
  "password": "wrongpassword"
}
```

Expected:

```json
false
```

!> **This `true`/`false` response is only for learning.** We won't keep it in the final API. In the next phase, a successful login returns an **access token** and a **refresh token** instead.

?> **What about an email that doesn't exist?** `orElseThrow` throws a plain `RuntimeException`, which our `GlobalExceptionHandler` doesn't handle, so you get `500 Internal Server Error`. That's another thing we'll clean up when login returns tokens. A good login API returns the **same** error for a wrong email and a wrong password, so attackers can't find out which emails are registered.

## 🎉 Phase 3 complete

You now have:

| Feature | How |
|---------|-----|
| Transactions | `@Transactional` and `@Transactional(readOnly = true)` in the service |
| Pagination | `Pageable` + `StudentPageResponse` |
| Sorting | `?sort=field,asc` or `?sort=field,desc` through the same `Pageable` |
| Security on | `spring-boot-starter-security` + `SecurityFilterChain` + Basic Auth |
| Users in the database | `User` entity → `users` table, `findByEmail()` |
| Safe signup | BCrypt hash, `UserResponse` without the password |
| Login check | `passwordEncoder.matches()` |

**Coming next:** generating **JWT access and refresh tokens** at login, validating them on every request, protecting the student endpoints, and adding **USER / ADMIN** roles.

## ✅ Checkpoint

- [ ] `dto/LoginRequest.java` exists
- [ ] `AuthService.login()` uses `passwordEncoder.matches()`
- [ ] Correct password → `true`, wrong password → `false`

Next: **[Final Code](phase-3/final-code.md)**: every changed file in its finished Phase 3 state →
