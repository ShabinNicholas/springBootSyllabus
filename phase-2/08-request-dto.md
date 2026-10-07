# 8. Request DTO

> Covers **Steps 29–29.8**: accept a `StudentRequest` DTO in POST instead of the `Student` entity.

## Why DTOs?

So far, the API uses the `Student` **entity** directly:

```java
@PostMapping
public ResponseEntity<Student> createStudent(
        @Valid @RequestBody Student student) {
```

This works, but real Spring Boot applications usually **don't expose the entity through the API**. Instead they use a **DTO (Data Transfer Object)**: a simple class that describes the data going in or out of the API.

```text
Client
  ↓
StudentRequest DTO   ← what the API accepts
  ↓
Controller
  ↓
Service              ← copies DTO values into the entity
  ↓
Student Entity       ← what the database stores
  ↓
Database
```

**Why bother?** Later, the `Student` entity might grow fields like:

```text
id, name, email, age, password, createdAt, updatedAt
```

You don't want clients to **send** an `id` or `createdAt`, or to **receive** a `password`. With DTOs, the entity can change without changing the API, and the API only accepts and returns the fields you choose.

## Step 29.1 — Create the dto package

Inside `src/main/java/com/example/student_api`, create a new folder called **`dto`**:

```text
com.example.student_api
├── controller
├── dto          ← new
├── entity
├── repository
└── service
```

## Step 29.2 — Create StudentRequest

Inside `dto`, create **`StudentRequest.java`**:

```java
package com.example.student_api.dto;

public class StudentRequest {

}
```

## Step 29.3 — Add the fields

The client should send only `name`, `email` and `age`. **No `id`**, because the database generates it.

```java
package com.example.student_api.dto;

public class StudentRequest {

    private String name;
    private String email;
    private Integer age;

}
```

## Step 29.4 — Add getters and setters

Just like the entity in Phase 1, Jackson needs getters and setters to read JSON into this class. Add them inside the class:

```java
public String getName() {
    return name;
}

public void setName(String name) {
    this.name = name;
}

public String getEmail() {
    return email;
}

public void setEmail(String email) {
    this.email = email;
}

public Integer getAge() {
    return age;
}

public void setAge(Integer age) {
    this.age = age;
}
```

## Step 29.5 — Add validation to the DTO

Validation is about **what the API accepts**, so it belongs on the request DTO. Add these imports:

```java
import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.Min;
import jakarta.validation.constraints.NotBlank;
```

and the same rules we used on the entity. The complete class:

```java
package com.example.student_api.dto;

import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.Min;
import jakarta.validation.constraints.NotBlank;

public class StudentRequest {

    @NotBlank(message = "Name is required")
    private String name;

    @NotBlank(message = "Email is required")
    @Email(message = "Email must be valid")
    private String email;

    @Min(value = 18, message = "Age must be at least 18")
    private Integer age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}
```

!> Don't change the controller or the `Student` entity yet.

## Step 29.6 — Use StudentRequest in POST

Open `StudentController.java` and add this import:

```java
import com.example.student_api.dto.StudentRequest;
```

Change only the **parameter type** of the POST method, from `Student` to `StudentRequest`:

```java
@PostMapping
public ResponseEntity<Student> createStudent(@Valid @RequestBody StudentRequest student) {
    Student createdStudent = studentService.createStudent(student);
    return ResponseEntity.status(201).body(createdStudent);
}
```

!> **There will be a red error** on `studentService.createStudent(student)`. That's expected: the service still expects a `Student`, but we're now passing a `StudentRequest`. We fix it next.

## Step 29.7 — Change createStudent() in the service

Open `StudentService.java` and add this import:

```java
import com.example.student_api.dto.StudentRequest;
```

Replace:

```java
public Student createStudent(Student student) {
    return studentRepository.save(student);
}
```

with:

```java
public Student createStudent(StudentRequest studentRequest) {

    Student student = new Student();

    student.setName(studentRequest.getName());
    student.setEmail(studentRequest.getEmail());
    student.setAge(studentRequest.getAge());

    return studentRepository.save(student);
}
```

The service now **builds a new entity** from the DTO's values, then saves it:

```text
POST request
   ↓
StudentRequest DTO
   ↓
Service
   ↓
new Student entity
   ↓
Save to PostgreSQL
```

The red error in the controller is gone. Don't change the PUT method yet.

## Step 29.8 — Test POST with the DTO

Restart the app:

```powershell
.\mvnw spring-boot:run
```

Send <span class="method post">POST</span> `http://localhost:8080/students`:

```json
{
  "name": "Sarah",
  "email": "sarah@gmail.com",
  "age": 23
}
```

Expected: **`201 Created`** with a new ID (10 in this example). From the outside nothing looks different, but inside, the request now goes:

```text
JSON → StudentRequest DTO → Validation → StudentService → Student Entity → PostgreSQL
```

?> Try an invalid request too (for example `"age": 15`). You still get `400` with the message, because the rules on `StudentRequest` are being checked now.

## ✅ Checkpoint

- [ ] `dto/StudentRequest.java` exists with fields, getters/setters and validation
- [ ] POST accepts `StudentRequest`
- [ ] The service builds a `Student` from the DTO
- [ ] POST still returns `201 Created`

Next: **[9. DTO for PUT](phase-2/09-request-dto-put.md)** →
