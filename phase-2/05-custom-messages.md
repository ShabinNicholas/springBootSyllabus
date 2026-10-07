# 5. Custom Messages

> Covers **Steps 27–27.3**: give each validation rule a human-readable message.

## What we'll do

Every validation annotation takes an optional `message`. That's the text we'll eventually show the client when the rule fails.

## Step 27 — Message for name

In `Student.java`, change:

```java
@NotBlank
private String name;
```

to:

```java
@NotBlank(message = "Name is required")
private String name;
```

## Step 27.1 — Message for a missing email

Change **only the `@NotBlank` line** on email:

```java
@NotBlank(message = "Email is required")
@Email
private String email;
```

## Step 27.2 — Message for age

Change:

```java
@Min(18)
private Integer age;
```

to:

```java
@Min(value = 18, message = "Age must be at least 18")
private Integer age;
```

?> With only one setting you can write `@Min(18)`. With two settings, you have to name them: `@Min(value = 18, message = "...")`.

Your validation section is now:

```java
@NotBlank(message = "Name is required")
private String name;

@NotBlank(message = "Email is required")
@Email
private String email;

@Min(value = 18, message = "Age must be at least 18")
private Integer age;
```

Save the file.

## Step 27.3 — Test it

Restart if needed, then send <span class="method post">POST</span> `http://localhost:8080/students` with **everything** wrong:

```json
{
  "name": "",
  "email": "",
  "age": 15
}
```

You get `400 Bad Request` again, but the body is still the generic one:

```json
{
  "timestamp": "...",
  "status": 400,
  "error": "Bad Request",
  "path": "/students"
}
```

**Where are our messages?** Spring creates them, but its default error response doesn't show them. When validation fails, Spring throws a `MethodArgumentNotValidException`, and the messages are inside it. To show them, we need to catch that exception ourselves.

## ✅ Checkpoint

- [ ] All three fields have custom messages
- [ ] Invalid data still returns `400`, without the messages yet

Next: **[6. Global Exception Handler](phase-2/06-exception-handler.md)** →
