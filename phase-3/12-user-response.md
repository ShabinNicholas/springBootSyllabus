# 12. Don't Return the Password

> Return a `UserResponse` DTO from signup, so the password hash never leaves the server.

## The problem

`AuthController` returns the whole `User` entity:

```java
return authService.signup(signupRequest);
```

That's why the hashed password shows up in the response. We'll fix this before moving on to login and tokens, using the same idea as `StudentResponse` in Phase 2.

## Step 1 — Create UserResponse

Create `src/main/java/com/example/student_api/dto/UserResponse.java` with **only** the safe fields, `id`, `name` and `email`. **No password.**

```java
package com.example.student_api.dto;

public class UserResponse {

    private Long id;
    private String name;
    private String email;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }
}
```

!> Only create the DTO for now. Don't change `AuthService` or `AuthController` yet.

## Step 2 — Return UserResponse from AuthService

Replace `AuthService.java` with:

```java
package com.example.student_api.service;

import com.example.student_api.dto.SignupRequest;
import com.example.student_api.dto.UserResponse;
import com.example.student_api.entity.User;
import com.example.student_api.repository.UserRepository;

import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;

@Service
public class AuthService {

    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;

    public AuthService(
            UserRepository userRepository,
            PasswordEncoder passwordEncoder) {

        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
    }

    public UserResponse signup(SignupRequest signupRequest) {

        User user = new User();

        user.setName(signupRequest.getName());
        user.setEmail(signupRequest.getEmail());

        String hashedPassword =
                passwordEncoder.encode(signupRequest.getPassword());

        user.setPassword(hashedPassword);

        User savedUser = userRepository.save(user);

        UserResponse response = new UserResponse();

        response.setId(savedUser.getId());
        response.setName(savedUser.getName());
        response.setEmail(savedUser.getEmail());

        return response;
    }
}
```

### The important change

We keep the saved user:

```java
User savedUser = userRepository.save(user);
```

then copy **only the safe fields** into a `UserResponse`:

```java
response.setId(savedUser.getId());
response.setName(savedUser.getName());
response.setEmail(savedUser.getEmail());
```

The password stays inside the entity and the database. It never leaves the service as part of the API response.

## Step 3 — Update the controller

`signup()` now returns `UserResponse`, so change the controller method:

```java
@PostMapping("/signup")
public UserResponse signup(@RequestBody SignupRequest signupRequest) {
    return authService.signup(signupRequest);
}
```

Add this import:

```java
import com.example.student_api.dto.UserResponse;
```

?> You can remove `import com.example.student_api.entity.User;` from `AuthController`, because the controller no longer uses it.

Restart the app.

## Step 4 — Test signup again

<span class="method post">POST</span> `http://localhost:8080/auth/signup`

```json
{
  "name": "Alice",
  "email": "alice@example.com",
  "password": "123456"
}
```

Expected:

```json
{
  "id": 2,
  "name": "Alice",
  "email": "alice@example.com"
}
```

**No `password` field.** ✅ In the `users` table, the password is still stored as a BCrypt hash.

```text
SignupRequest
   ↓
AuthController → AuthService → BCrypt PasswordEncoder
   ↓
User entity → PostgreSQL
   ↓
UserResponse
   ↓
Client (never sees the password or hash)
```

## ✅ Checkpoint

- [ ] `dto/UserResponse.java` exists with `id`, `name`, `email`
- [ ] `signup()` returns `UserResponse`
- [ ] The signup response has no `password` field

Next: **[13. Login](phase-3/13-login.md)** →
