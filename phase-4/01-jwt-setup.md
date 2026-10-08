# 1. JWT Setup

> Add the JJWT library and create an empty `JwtService`.

## Where we are

At the end of Phase 3, login checks the password and returns `true` or `false`:

```text
LoginRequest → find user by email → BCrypt hash from DB → passwordEncoder.matches() → true / false
```

Now we build the part we've been heading towards: **an access token and a refresh token**.

## What is a JWT?

A **JWT (JSON Web Token)** is a signed string the server gives the client after login. The client sends it back with every request, so the server knows who is calling without asking for the password again.

```text
HEADER.PAYLOAD.SIGNATURE
```

- **Header**: which algorithm signed the token
- **Payload**: information ("claims") such as the user's email and when the token expires
- **Signature**: proves the token was made by our server and hasn't been changed

!> The payload is only **encoded**, not encrypted. Anyone with the token can read it. Never put passwords or secrets inside a JWT.

## Step 1 — Add the JJWT dependencies

Open `pom.xml` and add these three dependencies inside `<dependencies>`:

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.13.0</version>
</dependency>

<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.13.0</version>
    <scope>runtime</scope>
</dependency>

<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.13.0</version>
    <scope>runtime</scope>
</dependency>
```

| Artifact | Purpose |
|----------|---------|
| `jjwt-api` | The classes we use in our code (`Jwts`, `Keys`, ...) |
| `jjwt-impl` | The implementation, needed only when the app runs |
| `jjwt-jackson` | Turns the token's JSON into Java objects and back |

?> Unlike the Spring Boot starters, these need a `<version>` because Spring Boot doesn't manage JJWT's version.

Save `pom.xml` and let VS Code reload the project. Don't create any JWT classes yet.

## Step 2 — Create JwtService

All token logic will live in one class. Create `src/main/java/com/example/student_api/service/JwtService.java`:

```java
package com.example.student_api.service;

import org.springframework.stereotype.Service;

@Service
public class JwtService {

    private final String secretKey =
            "my-super-secret-key-for-student-api-123456789";

}
```

The **secret key** is what we sign tokens with. Only the server knows it, so only the server can make valid tokens.

```text
AuthService → JwtService → generate access token / refresh token
```

Later `JwtService` will also have `validateToken()` and `extractEmail()`.

!> **For learning only:** the secret is hard-coded here. We move it to `application.properties` on [page 5](phase-4/05-secret-config.md).

Restart the app and make sure it starts.

## ✅ Checkpoint

- [ ] `pom.xml` has `jjwt-api`, `jjwt-impl` and `jjwt-jackson`
- [ ] `service/JwtService.java` exists and the app starts

Next: **[2. Access Token](phase-4/02-access-token.md)** →
