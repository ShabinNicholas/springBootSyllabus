# 7. Spring Security

> Authentication vs authorization, then add Spring Security and see what it does on its own.

## Step 1 — Authentication vs Authorization

These two words come up constantly. They mean different things.

### Authentication = "Who are you?"

```text
POST /auth/login
email:    shabin@gmail.com
password: ********
        ↓
Are these credentials correct?
        ↓
       YES
        ↓
User is authenticated
```

### Authorization = "What are you allowed to do?"

Once the server knows who you are, it decides what you can access. For example:

| Request | USER | ADMIN |
|---------|------|-------|
| <span class="method get">GET</span> `/students` | ✅ | ✅ |
| <span class="method post">POST</span> `/students` | ✅ | ✅ |
| <span class="method delete">DELETE</span> `/students/5` | ❌ | ✅ |

```text
Authentication → WHO are you?
Authorization  → WHAT can you do?
```

?> **How this connects to tokens later:** an **access token** will carry enough information for the server to identify the logged-in user and their roles. A **refresh token** is used to get a new access token when the old one expires. We build both in the next phase.

## Step 2 — Add the Spring Security dependency

Open `pom.xml` and add this inside `<dependencies>`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

Spring Boot will now **automatically** set up basic security for the whole application.

!> Don't create any security configuration class yet, and don't add JWT code. Just add the dependency.

## Step 3 — Restart

Stop the app (**Ctrl + C**) and run:

```powershell
.\mvnw spring-boot:run
```

Wait for `Started StudentApiApplication`. You'll also see a line like this in the console:

```text
Using generated security password: 6c3c1d97-664f-425a-a9ec-58c1f33d4c20
```

That's expected. We'll use it shortly.

## Step 4 — See what Spring Security did

Send <span class="method get">GET</span> `http://localhost:8080/students` with no credentials.

You **don't** get your students anymore. Depending on your client, you'll see one of these:

- **`401 Unauthorized`**, or
- Spring Security's default **login page** (HTML with *Username*, *Password* and *Sign in*). Clients like Bruno and browsers follow the redirect to `/login` and show this page.

Either way, Spring Security is now **active** and protecting your endpoints:

```text
Before:  Client → /students → StudentController → Student data
Now:     Client → /students → Spring Security → "Are you authenticated?" → NO → 🔒
```

As soon as you add the dependency, **every** endpoint requires login by default.

## Step 5 — Find the default user

Spring Boot created a default user for you:

- **Username:** `user`
- **Password:** the generated password from the console (`Using generated security password: ...`)

!> The generated password **changes every time the app restarts**. Always copy the newest one from the console.

## Step 6 — Try Basic Auth

In Bruno (or Postman), keep <span class="method get">GET</span> `http://localhost:8080/students`. Open the **Auth** tab and choose **Basic Auth**:

```text
Username: user
Password: <your generated password>
```

Send it. You'll probably get **`403 Forbidden`**, not your students.

### 401 vs 403

| Status | Meaning |
|--------|---------|
| `401 Unauthorized` | *"You are not authenticated."* (I don't know who you are) |
| `403 Forbidden` | *"I know who you are, but you're not allowed to do this."* |

Spring Security's default rules are still being applied, and `/students` isn't set up the way we want. On the next page we write our own rules with a `SecurityFilterChain`.

?> If you got your students back instead of `403`, that's fine too. Keep going: the next page puts the rules under **our** control either way.

## Where we're heading

We **won't** use this default login system in the final project. The goal is:

```text
Signup → email + password → BCrypt hash → Database

Login  → email + password → Access Token + Refresh Token → Protected APIs
```

We'll get there step by step.

## ✅ Checkpoint

- [ ] You can explain authentication vs authorization
- [ ] `spring-boot-starter-security` is in `pom.xml`
- [ ] `GET /students` without credentials is blocked (`401` or the login page)
- [ ] You found the generated password in the console

Next: **[8. SecurityConfig](phase-3/08-security-config.md)** →
