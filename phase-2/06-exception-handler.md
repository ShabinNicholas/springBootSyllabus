# 6. Global Exception Handler

> Covers **Steps 27.4–27.10**: catch validation errors and return clear messages.

## What we'll do

We'll create a class that handles exceptions for **every controller** in the app. When validation fails, it collects the field errors and returns them as JSON:

```json
{
  "name": "Name is required",
  "email": "Email is required",
  "age": "Age must be at least 18"
}
```

## Step 27.4 — Create the class

Inside `src/main/java/com/example/student_api/controller`, create **`GlobalExceptionHandler.java`**:

```java
package com.example.student_api.controller;

public class GlobalExceptionHandler {
}
```

Save it.

## Step 27.5 — Mark it as an exception handler

Add the import and the annotation:

```java
package com.example.student_api.controller;

import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
public class GlobalExceptionHandler {
}
```

`@RestControllerAdvice` means: *"This class gives advice to every `@RestController`. When one of them throws an exception, check here for a method that handles it."*

## Step 27.6 — Handle validation errors

Add this method inside the class:

```java
@ExceptionHandler(MethodArgumentNotValidException.class)
public Map<String, String> handleValidationErrors(
        MethodArgumentNotValidException ex) {

    Map<String, String> errors = new HashMap<>();

    ex.getBindingResult().getFieldErrors().forEach(error ->
            errors.put(error.getField(), error.getDefaultMessage())
    );

    return errors;
}
```

Your complete file:

```java
package com.example.student_api.controller;

import java.util.HashMap;
import java.util.Map;

import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public Map<String, String> handleValidationErrors(
            MethodArgumentNotValidException ex) {

        Map<String, String> errors = new HashMap<>();

        ex.getBindingResult().getFieldErrors().forEach(error ->
                errors.put(error.getField(), error.getDefaultMessage())
        );

        return errors;
    }
}
```

### What this means

| Code | Meaning |
|------|---------|
| `@ExceptionHandler(MethodArgumentNotValidException.class)` | Run this method when validation fails |
| `Map<String, String> errors` | A map of field name → message, which becomes a JSON object |
| `ex.getBindingResult().getFieldErrors()` | The list of fields that broke a rule |
| `error.getField()` | The field name, for example `"email"` |
| `error.getDefaultMessage()` | The `message` we wrote, for example `"Email is required"` |

Save the file.

## Step 27.7 — Test the messages

Restart the app, then send <span class="method post">POST</span> `http://localhost:8080/students`:

```json
{
  "name": "",
  "email": "",
  "age": 15
}
```

This time you get the messages:

```json
{
  "name": "Name is required",
  "email": "Email is required",
  "age": "Age must be at least 18"
}
```

?> The order of the fields may be different. A `HashMap` doesn't keep any order.

!> **Look at the status code in Postman.** You'll probably see **`200 OK`**, not `400`! An `@ExceptionHandler` method returns `200` unless we tell it otherwise. We fix this on [page 7](phase-2/07-validation-status.md).

## Step 27.8 — Message for an invalid email format

In `Student.java`, give `@Email` its own message:

```java
@NotBlank(message = "Email is required")
@Email(message = "Email must be valid")
private String email;
```

Save the file.

## Step 27.9 — Test the invalid email message

```json
{
  "name": "Test User",
  "email": "invalid-email",
  "age": 25
}
```

Expected:

```json
{
  "email": "Email must be valid"
}
```

Only the field that's wrong appears.

## Step 27.10 — Make sure valid data still works

```json
{
  "name": "Robert",
  "email": "robert@gmail.com",
  "age": 24
}
```

Expected: **`201 Created`**. The handler only runs when an exception happens, so normal requests aren't affected.

## The flow so far

```text
POST /students
   ↓
@Valid
   ↓
Student validation fails → MethodArgumentNotValidException
   ↓
GlobalExceptionHandler
   ↓
Clear validation messages
```

## ✅ Checkpoint

- [ ] `GlobalExceptionHandler` exists with `@RestControllerAdvice`
- [ ] Invalid data returns a JSON object of field messages
- [ ] Valid data still returns `201 Created`

Next: **[7. Validation Status Code](phase-2/07-validation-status.md)** →
