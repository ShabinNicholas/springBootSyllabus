# 7. Validation Status Code

> Covers **Step 28**: return `400 Bad Request` explicitly, then test every rule one by one.

## Step 28 — Add @ResponseStatus

Our handler returns the messages, but it doesn't say which HTTP status to use. Let's make it return **`400 Bad Request`** explicitly.

Open `GlobalExceptionHandler.java` and add these imports:

```java
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ResponseStatus;
```

?> You need both imports: `HttpStatus` provides `BAD_REQUEST`, and `ResponseStatus` is the annotation.

Add `@ResponseStatus(HttpStatus.BAD_REQUEST)` directly above the method:

```java
@ExceptionHandler(MethodArgumentNotValidException.class)
@ResponseStatus(HttpStatus.BAD_REQUEST)
public Map<String, String> handleValidationErrors(
        MethodArgumentNotValidException ex) {
```

Save the file and restart the app.

## Step 28.1 — Test the status

Send <span class="method post">POST</span> `http://localhost:8080/students` with invalid data:

```json
{
  "name": "",
  "email": "wrong-email",
  "age": 15
}
```

Expected: **`400 Bad Request`** with:

```json
{
  "name": "Name is required",
  "email": "Email must be valid",
  "age": "Age must be at least 18"
}
```

## Test each rule on its own

Testing one field at a time proves that each rule and message works independently.

### Step 28.2 — Missing email

```json
{
  "name": "Test Student",
  "email": "",
  "age": 25
}
```

Expected `400`:

```json
{
  "email": "Email is required"
}
```

So for email:

| Input | Result |
|-------|--------|
| Empty email | `"Email is required"` → `400` |
| Invalid email | `"Email must be valid"` → `400` |
| Valid email | Student created → `201` |

### Step 28.3 — Missing name

```json
{
  "name": "",
  "email": "test@gmail.com",
  "age": 25
}
```

Expected `400`:

```json
{
  "name": "Name is required"
}
```

### Step 28.4 — Age below 18

```json
{
  "name": "Test Student",
  "email": "test@gmail.com",
  "age": 17
}
```

Expected `400`:

```json
{
  "age": "Age must be at least 18"
}
```

### Step 28.5 — Validation messages on PUT

Use an existing student's ID (for example Robert, ID 9). Send <span class="method put">PUT</span> `http://localhost:8080/students/9`:

```json
{
  "name": "Robert Updated",
  "email": "wrong-email",
  "age": 25
}
```

Expected `400`:

```json
{
  "email": "Email must be valid"
}
```

PUT uses the same global handler. We didn't write any extra code for it.

## ✅ Validation complete

| Field | Rules | Message(s) |
|-------|-------|------------|
| `name` | `@NotBlank` | Name is required |
| `email` | `@NotBlank` + `@Email` | Email is required / Email must be valid |
| `age` | `@Min(18)` | Age must be at least 18 |

Your `Student.java` fields now look like:

```java
@NotBlank(message = "Name is required")
private String name;

@NotBlank(message = "Email is required")
@Email(message = "Email must be valid")
private String email;

@Min(value = 18, message = "Age must be at least 18")
private Integer age;
```

## ✅ Checkpoint

- [ ] Invalid data returns **`400 Bad Request`** (not `200`)
- [ ] Each rule returns its own message on POST and PUT

Next: **[8. Request DTO](phase-2/08-request-dto.md)** →
