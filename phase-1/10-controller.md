# 10. Controller & First GET

> Covers **Steps 13–15**: create the `StudentController`, run the backend and test the first API.

## What we'll do

The **controller** receives HTTP requests and sends back responses. We'll create our first endpoint:

<span class="method get">GET</span> `http://localhost:8080/students`

## Step 13.1 — Create the controller package

Inside `src/main/java/com/example/student_api`, create a new folder called **`controller`**.

## Step 13.2 — Create StudentController.java

Inside `controller`, create **`StudentController.java`**:

```java
package com.example.student_api.controller;

import com.example.student_api.entity.Student;
import com.example.student_api.service.StudentService;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
@RequestMapping("/students")
public class StudentController {

    private final StudentService studentService;

    public StudentController(StudentService studentService) {
        this.studentService = studentService;
    }

    @GetMapping
    public List<Student> getAllStudents() {
        return studentService.getAllStudents();
    }
}
```

## What this means

| Code | Meaning |
|------|---------|
| `@RestController` | This class handles HTTP requests, and what it returns is sent back as JSON |
| `@RequestMapping("/students")` | Every endpoint in this class starts with `/students` |
| `@GetMapping` | This method handles `GET /students` |
| Constructor | Spring injects the `StudentService` |

Save the file.

## Step 14 — Run the backend

Stop the app if it's running (**Ctrl + C**), then:

```powershell
.\mvnw spring-boot:run
```

Wait for:

```text
Started StudentApiApplication
```

Your server is now running on `http://localhost:8080`. Keep the terminal running.

## Step 15 — Test the GET API

Open this URL in your browser:

[http://localhost:8080/students](http://localhost:8080/students)

You should get:

```json
[]
```

An empty list is exactly what we expect, because we haven't added any students yet.

## ✅ Checkpoint

- [ ] `controller/StudentController.java` exists
- [ ] The app starts successfully
- [ ] `GET /students` returns `[]`

Next: **[11. Create API (POST)](phase-1/11-create-api.md)** →
