# 5. Secret Config

> Move the JWT secret out of the Java code and into `application.properties`.

## Why?

The secret is currently in `JwtService.java`:

```java
private final String secretKey =
        "my-super-secret-key-for-student-api-123456789";
```

Secrets shouldn't live in Java source code. Configuration can be changed per environment (your laptop, a test server, production) without touching the code.

## Step 1 — Add the property

Open `src/main/resources/application.properties` and add:

```properties
jwt.secret=my-super-secret-key-for-student-api-123456789
```

Restart and make sure the app starts. Don't change `JwtService` yet.

## Step 2 — Read it in JwtService

In `JwtService.java`, add this import:

```java
import org.springframework.beans.factory.annotation.Value;
```

Replace:

```java
private final String secretKey =
        "my-super-secret-key-for-student-api-123456789";
```

with:

```java
@Value("${jwt.secret}")
private String secretKey;
```

So the top of the class is:

```java
@Service
public class JwtService {

    @Value("${jwt.secret}")
    private String secretKey;

    // existing methods...
}
```

`@Value("${jwt.secret}")` tells Spring: *"fill this field with the `jwt.secret` value from the configuration."*

```text
application.properties → jwt.secret → JwtService → sign JWTs
```

?> Notice `final` is gone. Spring sets `@Value` fields **after** creating the object, so they can't be `final`.

Restart the app.

!> **Changing the secret logs everyone out.** Tokens are signed with the secret, so any token made with an old secret stops being valid. That's also why the secret must stay secret: anyone who knows it can create tokens for any user.

?> **Keep it out of GitHub.** `application.properties` is committed with your code. For a real project, read the value from an environment variable instead, for example `jwt.secret=${JWT_SECRET}`, and set `JWT_SECRET` on the machine that runs the app.

## ✅ Checkpoint

- [ ] `application.properties` has `jwt.secret`
- [ ] `JwtService` uses `@Value("${jwt.secret}")`
- [ ] Login still returns both tokens

Next: **[6. Validate a Token](phase-4/06-validate-token.md)** →
