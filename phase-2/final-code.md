# Phase 2 — Final Code

Every file that changed or was added in Phase 2, in its **finished state**. Files not listed here (`StudentApiApplication.java`, `StudentRepository.java`, `application.properties`) are the same as in the [Phase 1 Final Code](phase-1/final-code.md).

?> If your package name isn't `com.example.student_api`, change the `package` and `import` lines to match your project.

## Project structure

```text
student-api
├── src
│   └── main
│       ├── java
│       │   └── com.example.student_api
│       │       ├── StudentApiApplication.java
│       │       ├── controller
│       │       │   ├── GlobalExceptionHandler.java   ← new
│       │       │   └── StudentController.java
│       │       ├── dto                               ← new
│       │       │   ├── StudentRequest.java
│       │       │   └── StudentResponse.java
│       │       ├── entity
│       │       │   └── Student.java
│       │       ├── repository
│       │       │   └── StudentRepository.java
│       │       └── service
│       │           └── StudentService.java
│       └── resources
│           └── application.properties
└── pom.xml
```

## pom.xml (added dependency)

Inside `<dependencies>`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

## Database constraint

Run once on `student_db`:

```sql
ALTER TABLE student
ADD CONSTRAINT uk_student_email UNIQUE (email);
```

## entity/Student.java

```java
package com.example.student_api.entity;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;

@Entity
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @Column(nullable = false, unique = true)
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

## dto/StudentRequest.java

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

## dto/StudentResponse.java

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

## service/StudentService.java

```java
package com.example.student_api.service;

import com.example.student_api.dto.StudentRequest;
import com.example.student_api.dto.StudentResponse;
import com.example.student_api.entity.Student;
import com.example.student_api.repository.StudentRepository;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Service;
import org.springframework.web.server.ResponseStatusException;

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

    public Student createStudent(StudentRequest studentRequest) {

        Student student = new Student();

        student.setName(studentRequest.getName());
        student.setEmail(studentRequest.getEmail());
        student.setAge(studentRequest.getAge());

        return studentRepository.save(student);
    }

    public Student getStudentById(Long id) {
        return studentRepository.findById(id)
                .orElseThrow(() -> new ResponseStatusException(
                        HttpStatus.NOT_FOUND,
                        "Student not found"
                ));
    }

    public Student updateStudent(Long id, StudentRequest studentRequest) {

        Student existingStudent = studentRepository.findById(id)
                .orElseThrow(() -> new ResponseStatusException(
                        HttpStatus.NOT_FOUND,
                        "Student not found"
                ));

        existingStudent.setName(studentRequest.getName());
        existingStudent.setEmail(studentRequest.getEmail());
        existingStudent.setAge(studentRequest.getAge());

        return studentRepository.save(existingStudent);
    }

    public void deleteStudent(Long id) {
        Student student = studentRepository.findById(id)
                .orElseThrow(() -> new ResponseStatusException(
                        HttpStatus.NOT_FOUND,
                        "Student not found"
                ));

        studentRepository.delete(student);
    }

    public StudentResponse toStudentResponse(Student student) {

        StudentResponse response = new StudentResponse();

        response.setId(student.getId());
        response.setName(student.getName());
        response.setEmail(student.getEmail());
        response.setAge(student.getAge());

        return response;
    }
}
```

## controller/StudentController.java

```java
package com.example.student_api.controller;

import com.example.student_api.dto.StudentRequest;
import com.example.student_api.dto.StudentResponse;
import com.example.student_api.entity.Student;
import com.example.student_api.service.StudentService;
import jakarta.validation.Valid;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
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
    public List<StudentResponse> getAllStudents() {

        List<Student> students = studentService.getAllStudents();

        return students.stream()
                .map(student -> studentService.toStudentResponse(student))
                .toList();
    }

    @PostMapping
    public ResponseEntity<StudentResponse> createStudent(@Valid @RequestBody StudentRequest student) {

        Student createdStudent = studentService.createStudent(student);

        StudentResponse response = studentService.toStudentResponse(createdStudent);

        return ResponseEntity.status(201).body(response);
    }

    @GetMapping("/{id}")
    public StudentResponse getStudentById(@PathVariable Long id) {

        Student student = studentService.getStudentById(id);

        return studentService.toStudentResponse(student);
    }

    @PutMapping("/{id}")
    public ResponseEntity<StudentResponse> updateStudent(
            @PathVariable Long id,
            @Valid @RequestBody StudentRequest student) {

        Student updatedStudent = studentService.updateStudent(id, student);

        StudentResponse response = studentService.toStudentResponse(updatedStudent);

        return ResponseEntity.ok(response);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteStudent(@PathVariable Long id) {
        studentService.deleteStudent(id);

        return ResponseEntity.noContent().build();
    }
}
```

## controller/GlobalExceptionHandler.java

```java
package com.example.student_api.controller;

import java.util.HashMap;
import java.util.Map;

import org.springframework.dao.DataIntegrityViolationException;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import org.springframework.web.server.ResponseStatusException;

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public Map<String, String> handleValidationErrors(
            MethodArgumentNotValidException ex) {

        Map<String, String> errors = new HashMap<>();

        ex.getBindingResult().getFieldErrors().forEach(error ->
                errors.put(error.getField(), error.getDefaultMessage())
        );

        return errors;
    }

    @ExceptionHandler(ResponseStatusException.class)
    public ResponseEntity<Map<String, String>> handleResponseStatusException(
            ResponseStatusException ex) {

        Map<String, String> error = new HashMap<>();

        error.put("message", ex.getReason());

        return ResponseEntity
                .status(ex.getStatusCode())
                .body(error);
    }

    @ExceptionHandler(DataIntegrityViolationException.class)
    @ResponseStatus(HttpStatus.CONFLICT)
    public Map<String, String> handleDataIntegrityViolation(
            DataIntegrityViolationException ex) {

        Map<String, String> error = new HashMap<>();

        error.put("message", "Email already exists");

        return error;
    }
}
```

## API quick reference

| Operation | Method | Endpoint | Body | Success | Errors |
|-----------|--------|----------|------|---------|--------|
| Create | <span class="method post">POST</span> | `/students` | `StudentRequest` | `201` + `StudentResponse` | `400`, `409` |
| Read all | <span class="method get">GET</span> | `/students` | — | `200` + list | — |
| Read one | <span class="method get">GET</span> | `/students/{id}` | — | `200` + `StudentResponse` | `404` |
| Update | <span class="method put">PUT</span> | `/students/{id}` | `StudentRequest` | `200` + `StudentResponse` | `400`, `404`, `409` |
| Delete | <span class="method delete">DELETE</span> | `/students/{id}` | — | `204` | `404` |

Error response shapes:

```json
// 400 Bad Request: one entry per invalid field
{ "name": "Name is required", "email": "Email must be valid", "age": "Age must be at least 18" }

// 404 Not Found
{ "message": "Student not found" }

// 409 Conflict
{ "message": "Email already exists" }
```

## What's next?

Continue with **[Phase 3 — Transactions, Paging & Security Basics](phase-3/README.md)**: `@Transactional`, pagination, sorting, Spring Security, and BCrypt signup and login. 🚀
