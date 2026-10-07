# 10. Signup with BCrypt

> Create `SignupRequest`, an `AuthService`, and hash passwords with BCrypt.

## Step 1 — Create the SignupRequest DTO

Just like `StudentRequest`, the controller shouldn't accept the `User` entity directly. Create `src/main/java/com/example/student_api/dto/SignupRequest.java`:

```java
package com.example.student_api.dto;

public class SignupRequest {

    private String name;

    private String email;

    private String password;

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

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

The client will send:

```json
{
  "name": "Shabin",
  "email": "shabin@gmail.com",
  "password": "mypassword"
}
```

and it will travel like this:

```text
SignupRequest → AuthService → BCrypt hashing → User → PostgreSQL
```

?> **No validation yet.** We'll add rules like `@NotBlank` and `@Email` later, after the basic signup flow works.

Restart and make sure the app still starts.

## Step 2 — Create AuthService (first version)

`AuthService` will eventually handle signup, login, token generation and refresh tokens. For now, only signup.

Create `src/main/java/com/example/student_api/service/AuthService.java`:

```java
package com.example.student_api.service;

import com.example.student_api.dto.SignupRequest;
import com.example.student_api.entity.User;
import com.example.student_api.repository.UserRepository;

import org.springframework.stereotype.Service;

@Service
public class AuthService {

    private final UserRepository userRepository;

    public AuthService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public User signup(SignupRequest signupRequest) {

        User user = new User();

        user.setName(signupRequest.getName());
        user.setEmail(signupRequest.getEmail());
        user.setPassword(signupRequest.getPassword());

        return userRepository.save(user);
    }
}
```

```text
SignupRequest → AuthService → new User → UserRepository → PostgreSQL
```

!> **This version saves the password as plain text.** It's only here to show the basic flow. **Never do this in a real application.** We fix it in the next two steps, **before** we create the signup endpoint.

Restart and make sure the app starts. Don't create the controller yet.

## Step 3 — Create a PasswordEncoder

We'll use **BCrypt** to hash passwords. Create `src/main/java/com/example/student_api/config/PasswordConfig.java`:

```java
package com.example.student_api.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
public class PasswordConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

`@Bean` makes Spring create one `PasswordEncoder` that we can inject anywhere, just like our repositories.

### Why BCrypt?

Instead of storing:

```text
password = "123456"
```

the database stores something like:

```text
$2a$10$4sqD7ZTK/vQNJ95MBgeXROyPaCfjTSAG9PWvsOymcYmDyosq9fV4W
```

BCrypt is a **one-way hash**. You can't turn the hash back into the password, even if someone steals the database. At login, we don't decrypt anything: we hash the password the user typed and **compare** it with the stored hash.

?> BCrypt also adds a random **salt**, so hashing `123456` twice gives two **different** hashes. That's normal, and `matches()` (on the login page) still knows they belong to the same password.

Restart and make sure the app starts.

## Step 4 — Hash the password in AuthService

Replace `AuthService.java` with:

```java
package com.example.student_api.service;

import com.example.student_api.dto.SignupRequest;
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

    public User signup(SignupRequest signupRequest) {

        User user = new User();

        user.setName(signupRequest.getName());
        user.setEmail(signupRequest.getEmail());

        String hashedPassword =
                passwordEncoder.encode(signupRequest.getPassword());

        user.setPassword(hashedPassword);

        return userRepository.save(user);
    }
}
```

### The important change

```java
// Before
user.setPassword(signupRequest.getPassword());

// Now
String hashedPassword =
        passwordEncoder.encode(signupRequest.getPassword());

user.setPassword(hashedPassword);
```

```text
User enters password
   ↓
SignupRequest
   ↓
BCrypt PasswordEncoder
   ↓
Hashed password
   ↓
users table
```

Restart and make sure the app starts. Don't create the controller yet.

## ✅ Checkpoint

- [ ] `dto/SignupRequest.java` exists
- [ ] `config/PasswordConfig.java` provides a `BCryptPasswordEncoder` bean
- [ ] `AuthService.signup()` hashes the password before saving

Next: **[11. Signup Endpoint](phase-3/11-signup-endpoint.md)** →
