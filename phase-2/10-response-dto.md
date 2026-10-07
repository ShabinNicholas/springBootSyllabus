# 10. Response DTO

> Covers **Steps 30–30.6**: return a `StudentResponse` DTO from POST instead of the entity.

## Why a response DTO?

Requests now go through `StudentRequest`, but responses still return the entity directly:

```java
ResponseEntity<Student>
```

Today the entity's fields are harmless:

```json
{
  "id": 10,
  "name": "Sarah Updated",
  "email": "sarahupdated@gmail.com",
  "age": 24
}
```

But if `Student` later gets a `password` field, returning the entity would **leak it to every client**. A `StudentResponse` DTO lists exactly what the API returns, and nothing else.

## Step 30.1 — Create StudentResponse

Inside the `dto` package, create **`StudentResponse.java`**:

```java
package com.example.student_api.dto;

public class StudentResponse {

}
```

## Step 30.2 — Add the fields

Unlike the request, the response **does** include the `id`:

```java
package com.example.student_api.dto;

public class StudentResponse {

    private Long id;
    private String name;
    private String email;
    private Integer age;

}
```

## Step 30.3 — Add getters and setters

The complete class:

```java
package com.example.student_api.dto;

public class StudentResponse {

    private Long id;
    private String name;
    private String email;
    private Integer age;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

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

?> No validation annotations here. The response is data **we** produce, not data we receive, so there's nothing to validate.

## Step 30.4 — Convert Student → StudentResponse

We'll put the conversion in the service for now. Open `StudentService.java` and add this import:

```java
import com.example.student_api.dto.StudentResponse;
```

Add this method **at the bottom of the class**, before the final `}`:

```java
public StudentResponse toStudentResponse(Student student) {

    StudentResponse response = new StudentResponse();

    response.setId(student.getId());
    response.setName(student.getName());
    response.setEmail(student.getEmail());
    response.setAge(student.getAge());

    return response;
}
```

It copies values from the entity into a new response object:

```text
Student (entity)                StudentResponse (DTO)
id    = 10                 →    id    = 10
name  = Sarah Updated      →    name  = Sarah Updated
email = sarahupdated@...   →    email = sarahupdated@...
age   = 24                 →    age   = 24
```

!> Don't change the controller yet.

## Step 30.5 — Return StudentResponse from POST

Open `StudentController.java` and add this import:

```java
import com.example.student_api.dto.StudentResponse;
```

Replace the POST method:

```java
@PostMapping
public ResponseEntity<Student> createStudent(@Valid @RequestBody StudentRequest student) {
    Student createdStudent = studentService.createStudent(student);
    return ResponseEntity.status(201).body(createdStudent);
}
```

with:

```java
@PostMapping
public ResponseEntity<StudentResponse> createStudent(@Valid @RequestBody StudentRequest student) {

    Student createdStudent = studentService.createStudent(student);

    StudentResponse response = studentService.toStudentResponse(createdStudent);

    return ResponseEntity.status(201).body(response);
}
```

What changed:

```text
Before:  Student Entity → API response
After:   Student Entity → StudentResponse → API response
```

Don't change PUT or GET yet.

## Step 30.6 — Test POST

Restart the app and send <span class="method post">POST</span> `http://localhost:8080/students`:

```json
{
  "name": "Kevin",
  "email": "kevin@gmail.com",
  "age": 26
}
```

Expected **`201 Created`**:

```json
{
  "id": 11,
  "name": "Kevin",
  "email": "kevin@gmail.com",
  "age": 26
}
```

The JSON looks the same as before, but it now comes from `StudentResponse` instead of the entity.

!> **Send this request only once.** We haven't added duplicate-email protection yet, so sending it twice creates two Kevins with the same email. That matters on [page 13](phase-2/13-duplicate-emails.md). If it already happened, don't worry: page 13 shows how to clean it up.

## ✅ Checkpoint

- [ ] `dto/StudentResponse.java` exists with `id`, `name`, `email`, `age`
- [ ] The service has `toStudentResponse()`
- [ ] POST returns `ResponseEntity<StudentResponse>` with `201 Created`

Next: **[11. Response DTO Everywhere](phase-2/11-response-dto-all.md)** →
