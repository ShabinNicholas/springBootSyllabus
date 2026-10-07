# 11. Signup Endpoint

> Expose `POST /auth/signup`, fix the `403` caused by CSRF, and test it.

## Step 1 — Create AuthController

Password hashing is ready, so it's now safe to expose signup. Create `src/main/java/com/example/student_api/controller/AuthController.java`:

```java
package com.example.student_api.controller;

import com.example.student_api.dto.SignupRequest;
import com.example.student_api.entity.User;
import com.example.student_api.service.AuthService;

import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/auth")
public class AuthController {

    private final AuthService authService;

    public AuthController(AuthService authService) {
        this.authService = authService;
    }

    @PostMapping("/signup")
    public User signup(@RequestBody SignupRequest signupRequest) {
        return authService.signup(signupRequest);
    }
}
```

?> `import org.springframework.web.bind.annotation.*;` imports every annotation in that package (`@RestController`, `@PostMapping`, `@RequestBody`...) in one line.

```text
POST /auth/signup
   ↓
AuthController
   ↓
AuthService
   ↓
BCrypt hashes password
   ↓
UserRepository
   ↓
PostgreSQL users table
```

Restart the app. Don't test yet.

## Step 2 — Test signup (see the problem)

In Bruno, create a request:

- **Method:** `POST`
- **URL:** `http://localhost:8080/auth/signup`
- **Body → JSON:**

```json
{
  "name": "Shabin",
  "email": "shabin@example.com",
  "password": "123456"
}
```

Don't add any Authorization.

You get **`403 Forbidden`**. 🤔

### Why 403?

Look at our `SecurityConfig`:

```java
.requestMatchers("/students").authenticated()
.anyRequest().permitAll()
```

`/auth/signup` falls under `anyRequest().permitAll()`, so it *should* be allowed. But Spring Security has **CSRF protection turned on by default**, and it rejects `POST`, `PUT` and `DELETE` requests that don't carry a CSRF token.

?> **CSRF (Cross-Site Request Forgery)** is an attack where another website tricks your browser into sending a request using your login cookie. The protection matters for browser apps that use **cookie/session** login.

## Step 3 — Disable CSRF for our REST API

Our API will use **JWT access tokens** sent in a header, not cookies, so we'll disable CSRF. Change `SecurityConfig.java` to:

```java
package com.example.student_api.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {

        http
                .csrf(csrf -> csrf.disable())
                .authorizeHttpRequests(auth -> auth
                        .requestMatchers("/students").authenticated()
                        .anyRequest().permitAll()
                )
                .httpBasic(customizer -> {});

        return http.build();
    }
}
```

The only new line is:

```java
.csrf(csrf -> csrf.disable())
```

```text
POST /auth/signup → CSRF disabled → permitAll() → AuthController → AuthService → BCrypt → Database
```

Restart the app.

?> This also explains something from earlier: with CSRF on, `POST`, `PUT` and `DELETE` on student endpoints would be rejected too. Disabling it lets them work again.

## Step 4 — Test signup again

Send the same request:

<span class="method post">POST</span> `http://localhost:8080/auth/signup`

```json
{
  "name": "Shabin",
  "email": "shabin@example.com",
  "password": "123456"
}
```

Now you get **`200 OK`**:

```json
{
  "id": 1,
  "name": "Shabin",
  "email": "shabin@example.com",
  "password": "$2a$10$4sqD7ZTK/vQNJ95MBgeXROyPaCfjTSAG9PWvsOymcYmDyosq9fV4W"
}
```

The password is a **BCrypt hash** starting with `$2a$10$`, **not** `"123456"`. ✅

You can check the database too:

```sql
SELECT * FROM users;
```

!> **One problem:** we're returning the `User` entity directly, so the hashed password appears in the response. That's bad practice, because even a hash shouldn't leave the server. We fix it on the next page.

?> **Signing up twice with the same email** creates two users, because nothing stops it yet. That will break login later (`findByEmail` expects one result). If it happens, delete the extra row in psql. Adding `@Column(unique = true)` to `email` in `User`, like we did for students in Phase 2, is a good improvement to make later.

## ✅ Checkpoint

- [ ] `controller/AuthController.java` has `POST /auth/signup`
- [ ] `SecurityConfig` disables CSRF
- [ ] Signup returns `200` with a hashed password

Next: **[12. Don't Return the Password](phase-3/12-user-response.md)** →
