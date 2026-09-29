# 15. 404 Not Found

> Covers **Step 23**: return `404 Not Found` when a student doesn't exist.

## The problem

Right now, if you ask for a student that doesn't exist:

<span class="method get">GET</span> `/students/1` (a student we've already deleted)

the service does `.orElse(null)`, so the API returns `200 OK` with an **empty body**. That's misleading: `200 OK` says *"everything worked"*, but there's no student.

A good REST API returns **`404 Not Found`** instead.

## Step 23.2 — Throw a 404 from the service

Open `StudentService.java` and add these imports:

```java
import org.springframework.http.HttpStatus;
import org.springframework.web.server.ResponseStatusException;
```

?> Both imports are needed: `ResponseStatusException` is the exception, and `HttpStatus` provides `NOT_FOUND`.

Replace the current `getStudentById()` method:

```java
public Student getStudentById(Long id) {
    return studentRepository.findById(id).orElse(null);
}
```

with:

```java
public Student getStudentById(Long id) {
    return studentRepository.findById(id)
            .orElseThrow(() -> new ResponseStatusException(
                    HttpStatus.NOT_FOUND,
                    "Student not found"
            ));
}
```

## What changed?

| Before | After |
|--------|-------|
| `.orElse(null)` | `.orElseThrow(...)` |
| *"If the student doesn't exist, return `null`."* | *"If the student doesn't exist, throw an exception."* |

`ResponseStatusException` with `HttpStatus.NOT_FOUND` tells Spring Boot to stop and send back **`404 Not Found`**, with a message saying the student wasn't found.

!> Don't test it yet. Make this change and save the file first.

## Step 23.3 — Test the 404 response

Restart the app if needed. Use an ID that doesn't exist (for example a student you've deleted). In Postman:

- **Method:** `GET`
- **URL:** `http://localhost:8080/students/1`

This time, instead of an empty `200 OK`, you should get **`404 Not Found`** with a body like:

```json
{
  "timestamp": "...",
  "status": 404,
  "error": "Not Found",
  "path": "/students/1"
}
```

?> **Where's the "Student not found" message?** By default Spring Boot **hides** error messages from the response, so they don't leak internal details. The fields you see depend on your Spring Boot version and settings. If you want the message while learning, add this line to `application.properties`: `server.error.include-message=always`

## ✅ Checkpoint

- [ ] `StudentService` imports `HttpStatus` and `ResponseStatusException`
- [ ] `getStudentById()` uses `orElseThrow`
- [ ] `GET /students/{missing-id}` returns `404 Not Found`

Next: **[16. HTTP Status Codes](phase-1/16-status-codes.md)** →
