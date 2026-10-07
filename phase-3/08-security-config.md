# 8. SecurityConfig

> Write our own security rules with `SecurityFilterChain`, use Basic Auth, and see where the default user comes from.

## Step 1 — Create SecurityConfig

We need to tell Spring Security **which endpoints are open** and **which need a logged-in user**.

Create a new package **`config`** inside `com.example.student_api`, and inside it create **`SecurityConfig.java`**:

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
                .authorizeHttpRequests(auth -> auth
                        .requestMatchers("/students").authenticated()
                        .anyRequest().permitAll()
                )
                .httpBasic(customizer -> {});

        return http.build();
    }
}
```

### What this says

| Code | Meaning |
|------|---------|
| `@Configuration` + `@Bean` | Spring uses this `SecurityFilterChain` instead of the default rules |
| `.requestMatchers("/students").authenticated()` | `/students` needs a logged-in user |
| `.anyRequest().permitAll()` | Everything else is open, **for now** |
| `.httpBasic(customizer -> {})` | Log in with username + password using **HTTP Basic Auth**, with the default settings |

!> **Spring Security 7 (Spring Boot 4):** older tutorials write `.httpBasic()` with no arguments. That version has been removed. Don't pass `null` either: `.httpBasic(null)` makes the app **fail to start** with `Cannot invoke "Customizer.customize(Object)" because "httpBasicCustomizer" is null`. Use `.httpBasic(customizer -> {})`, or the equivalent `.httpBasic(Customizer.withDefaults())` (with `import org.springframework.security.config.Customizer;`).

!> **`"/students"` only matches that exact path.** `/students/5` is **not** covered, so with this config `GET /students/5`, `PUT` and `DELETE` work without logging in. To protect every student URL, you'd use `.requestMatchers("/students/**")`. We keep the simple version for this learning step; the next phase protects the student endpoints properly.

We're **not** implementing JWT yet. First we learn this basic flow:

```text
Request
   ↓
SecurityFilterChain
   ↓
Does this URL need authentication?
   ↓
Username + Password (Basic Auth)
   ↓
Authenticated ✅
   ↓
Controller
```

## Step 2 — Restart

Stop the app (**Ctrl + C**) and run `.\mvnw spring-boot:run`. Wait for `Started StudentApiApplication`.

## Step 3 — Test Basic Auth again

In Bruno, send <span class="method get">GET</span> `http://localhost:8080/students` with **Auth → Basic Auth**:

```text
Username: user
Password: <the NEW generated password from this startup>
```

!> Use the password from your **latest** startup, not the old one.

This time you get your students:

```json
{
  "content": [
    {
      "id": 3,
      "name": "Shabin",
      "email": "shabin@gmail.com",
      "age": 24
    }
  ],
  "page": 0,
  "size": 20,
  "totalElements": 10,
  "totalPages": 1
}
```

?> `size` is 20 because we didn't send `?size=`, so Spring used its default page size.

```text
Bruno
  │  GET /students
  ▼
Spring Security
  │  Username + Password
  ▼
Authentication ✅
  ▼
StudentController → StudentService → PostgreSQL
  ▼
Students JSON ✅
```

Our rule `.requestMatchers("/students").authenticated()` means **any** logged-in user can access `/students`. Later we'll add roles:

```text
USER  → can view students
ADMIN → can create, update and delete students
```

## Step 4 — Where does the default user come from?

We never created a `User` entity, repository or password. So where did `user` and its password come from?

When Spring Boot sees `spring-boot-starter-security` and you **haven't provided your own way to load users**, it creates an **in-memory user** automatically. Your startup log shows it:

```text
Global AuthenticationManager configured with
UserDetailsService bean with name inMemoryUserDetailsManager
```

**`inMemoryUserDetailsManager`** means the user lives in **application memory**, not in PostgreSQL. That's why the password changes on every restart.

```text
Spring Boot starts
   ↓
Creates default user (username = user, password = generated)
   ↓
Stores it in memory
   ↓
Spring Security uses it to check Basic Auth
```

## Why we don't want this

A real application stores users in the **database**:

```text
users table
├── id
├── name
├── email
├── password   (BCrypt hash)
└── role
```

and logs them in like this:

```text
Login → find user by email → compare password with BCrypt
      → generate Access Token + Refresh Token
```

```text
Current:  Spring Security → in-memory default user
Final:    Spring Security → database users + BCrypt + JWT access/refresh tokens
```

Next, we create our own `User` entity.

## ✅ Checkpoint

- [ ] `config/SecurityConfig.java` exists and the app starts
- [ ] `GET /students` with Basic Auth (`user` + the new password) returns your students
- [ ] You know the default user is stored in memory

Next: **[9. User Entity](phase-3/09-user-entity.md)** →
