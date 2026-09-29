# 9. Service Layer

> Covers **Step 12**: create the `StudentService`.

## What we'll do

Now we'll create the **service layer**. This layer holds the logic for working with students. The controller (next page) calls the service, and the service calls the repository.

```text
Controller  →  Service  →  Repository  →  Database
```

## Step 12.1 — Create the service package

Inside `src/main/java/com/example/student_api`, create a new folder called **`service`**.

## Step 12.2 — Create StudentService.java

Inside `service`, create **`StudentService.java`**:

```java
package com.example.student_api.service;

import com.example.student_api.entity.Student;
import com.example.student_api.repository.StudentRepository;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class StudentService {

    private final StudentRepository studentRepository;

    public StudentService(StudentRepository studentRepository) {
        this.studentRepository = studentRepository;
    }

    public List<Student> getAllStudents() {
        return studentRepository.findAll();
    }
}
```

## What this means

For now we're only creating the **Read** operation: `getAllStudents()`. It calls `studentRepository.findAll()` to get all students from PostgreSQL.

| Code | Meaning |
|------|---------|
| `@Service` | Tells Spring to create and manage this class (a Spring **bean**) |
| `private final StudentRepository studentRepository;` | The service needs a repository to do its work |
| The constructor | **Constructor injection**: Spring passes in the `StudentRepository` it created for us |

?> We never write `new StudentRepository()`. Spring creates the objects and connects them for us. This is called **dependency injection**.

!> **Do only this step.** Save the file.

## ✅ Checkpoint

- [ ] `service/StudentService.java` exists with `getAllStudents()`

Next: **[10. Controller & First GET](phase-1/10-controller.md)** →
