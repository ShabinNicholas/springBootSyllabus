# 3. Email & Age Validation

> Covers **Steps 26.6–26.11**: validate the email format and a minimum age.

## Step 26.6 — Add email validation

Open `Student.java` and add this import:

```java
import jakarta.validation.constraints.Email;
```

Find:

```java
private String email;
```

Change it to:

```java
@Email
private String email;
```

So the fields are now:

```java
@NotBlank
private String name;

@Email
private String email;

private Integer age;
```

`@Email` checks that the value **looks like an email address** (something like `user@domain`).

Save the file. Don't add anything else yet.

## Step 26.7 — Test an invalid email

Restart the app if needed, then send <span class="method post">POST</span> `http://localhost:8080/students`:

```json
{
  "name": "Test User",
  "email": "invalid-email",
  "age": 22
}
```

Expected: **`400 Bad Request`**, because `"invalid-email"` isn't a valid email format.

## Step 26.8 — Test a valid email

```json
{
  "name": "Test User",
  "email": "testuser@gmail.com",
  "age": 22
}
```

Expected: **`201 Created`**.

## Step 26.9 — Add age validation

We'll require students to be **at least 18**. Add this import to `Student.java`:

```java
import jakarta.validation.constraints.Min;
```

Find:

```java
private Integer age;
```

Change it to:

```java
@Min(18)
private Integer age;
```

Your validation section is now:

```java
@NotBlank
private String name;

@Email
private String email;

@Min(18)
private Integer age;
```

Save the file.

## Step 26.10 — Test an invalid age

```json
{
  "name": "Young User",
  "email": "young@gmail.com",
  "age": 16
}
```

Expected: **`400 Bad Request`**, because 16 is less than 18.

## Step 26.11 — Test a valid age

```json
{
  "name": "Michael",
  "email": "michael@gmail.com",
  "age": 25
}
```

Expected: **`201 Created`**. Note Michael's `id`. We'll use it on the next page.

## Validation so far

| Field | Rule | Meaning |
|-------|------|---------|
| `name` | `@NotBlank` | Required, can't be empty or spaces |
| `email` | `@Email` | Must look like an email |
| `age` | `@Min(18)` | Must be 18 or more |

?> **What if `age` is missing?** `@Min` only checks a value that is there. If the JSON has no `age` at all, it's `null` and `@Min` lets it through. If age should be required, you'd also add `@NotNull`. We don't need that for this guide.

## ✅ Checkpoint

- [ ] Invalid email → `400`, valid email → `201`
- [ ] Age 16 → `400`, age 25 → `201`

Next: **[4. Validate PUT](phase-2/04-validate-put.md)** →
