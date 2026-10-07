# 12. Clean 404 Messages

> Covers **Step 30.10**: handle `ResponseStatusException` in the global handler.

## The problem

When you request a student that doesn't exist:

<span class="method get">GET</span> `http://localhost:8080/students/999`

you get `404`, but with Spring's default error body. As we saw in Phase 1, that body **hides our "Student not found" message**:

```json
{
  "timestamp": "...",
  "status": 404,
  "error": "Not Found",
  "path": "/students/999"
}
```

We already have a `GlobalExceptionHandler`, so let's teach it to handle `ResponseStatusException` too, the exception our service throws for missing students.

## Step 30.10 — Add the handler method

Open `GlobalExceptionHandler.java` and add these imports:

```java
import org.springframework.http.ResponseEntity;
import org.springframework.web.server.ResponseStatusException;
```

Add this method inside the class:

```java
@ExceptionHandler(ResponseStatusException.class)
public ResponseEntity<Map<String, String>> handleResponseStatusException(
        ResponseStatusException ex) {

    Map<String, String> error = new HashMap<>();

    error.put("message", ex.getReason());

    return ResponseEntity
            .status(ex.getStatusCode())
            .body(error);
}
```

### What this means

| Code | Meaning |
|------|---------|
| `@ExceptionHandler(ResponseStatusException.class)` | Run this when any `ResponseStatusException` is thrown |
| `ex.getReason()` | The message we passed in the service: `"Student not found"` |
| `ex.getStatusCode()` | The status we passed in the service: `NOT_FOUND` |
| `ResponseEntity.status(...).body(...)` | Keeps the original status, so it doesn't become `200` or `400` |

?> Why `ResponseEntity` here instead of `@ResponseStatus`? `@ResponseStatus` sets **one fixed** status. A `ResponseStatusException` can carry **any** status (404, 409, 403...), so we read it from the exception and pass it on.

Your handler now handles two kinds of error:

1. **Validation errors** → `400 Bad Request`
2. **Student not found** → `404 Not Found`

## Test

Restart the app. Send <span class="method get">GET</span> `http://localhost:8080/students/999`.

Expected **`404 Not Found`**:

```json
{
  "message": "Student not found"
}
```

?> `PUT /students/999` and `DELETE /students/999` now return the same clean message, because they throw the same exception in the service.

## ✅ Checkpoint

- [ ] `GlobalExceptionHandler` has a `ResponseStatusException` handler
- [ ] A missing student returns `404` with `{"message": "Student not found"}`

Next: **[13. Duplicate Emails](phase-2/13-duplicate-emails.md)** →
