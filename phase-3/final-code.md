# Phase 3 — Final Code

Every file that changed or was added in Phase 3, in its **finished state**. Files not listed here are the same as in the [Phase 2 Final Code](phase-2/final-code.md): `StudentApiApplication.java`, `Student.java`, `StudentRepository.java`, `StudentRequest.java`, `StudentResponse.java`, `GlobalExceptionHandler.java` and `application.properties`.

?> If your package name isn't `com.example.student_api`, change the `package` and `import` lines to match your project.

## Project structure

```text
com.example.student_api
├── StudentApiApplication.java
├── config                         ← new
│   ├── PasswordConfig.java
│   └── SecurityConfig.java
├── controller
│   ├── AuthController.java        ← new
│   ├── GlobalExceptionHandler.java
│   └── StudentController.java     (changed)
├── dto
│   ├── LoginRequest.java          ← new
│   ├── SignupRequest.java         ← new
│   ├── StudentPageResponse.java   ← new
│   ├── StudentRequest.java
│   ├── StudentResponse.java
│   └── UserResponse.java          ← new
├── entity
│   ├── Student.java
│   └── User.java                  ← new
├── repository
│   ├── StudentRepository.java
│   └── UserRepository.java        ← new
└── service
    ├── AuthService.java           ← new
    └── StudentService.java        (changed)
```

## pom.xml (added dependency)

Inside `<dependencies>`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

## config/SecurityConfig.java

```java
package com.example.student_api.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {

        http
                .csrf(csrf -> csrf.disable())
                .authorizeHttpRequests(auth -> auth
                        .requestMatchers("/students").authenticated()
                        .anyRequest().permitAll()
                )
                .httpBasic(customizer -> {});

        return http.build();
    }
}
```

!> Remember: `"/students"` protects only that exact path. `/students/{id}` is still open at this stage. The next phase protects the student endpoints properly.

## config/PasswordConfig.java

```java
package com.example.student_api.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
public class PasswordConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

## entity/User.java

```java
package com.example.student_api.entity;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.Table;

@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private String email;

    private String password;

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

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

## repository/UserRepository.java

```java
package com.example.student_api.repository;

import com.example.student_api.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.Optional;

public interface UserRepository extends JpaRepository<User, Long> {

    Optional<User> findByEmail(String email);

}
```

## dto/StudentPageResponse.java

```java
package com.example.student_api.dto;

import java.util.List;

public class StudentPageResponse {

    private List<StudentResponse> content;
    private int page;
    private int size;
    private long totalElements;
    private int totalPages;

    public List<StudentResponse> getContent() {
        return content;
    }

    public void setContent(List<StudentResponse> content) {
        this.content = content;
    }

    public int getPage() {
        return page;
    }

    public void setPage(int page) {
        this.page = page;
    }

    public int getSize() {
        return size;
    }

    public void setSize(int size) {
        this.size = size;
    }

    public long getTotalElements() {
        return totalElements;
    }

    public void setTotalElements(long totalElements) {
        this.totalElements = totalElements;
    }

    public int getTotalPages() {
        return totalPages;
    }

    public void setTotalPages(int totalPages) {
        this.totalPages = totalPages;
    }
}
```

## dto/SignupRequest.java

```java
package com.example.student_api.dto;

public class SignupRequest {

    private String name;

    private String email;

    private String password;

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

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

## dto/LoginRequest.java

```java
package com.example.student_api.dto;

public class LoginRequest {

    private String email;

    private String password;

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

## dto/UserResponse.java

```java
package com.example.student_api.dto;

public class UserResponse {

    private Long id;
    private String name;
    private String email;

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
}
```

## service/StudentService.java

```java
package com.example.student_api.service;

import com.example.student_api.dto.StudentPageResponse;
import com.example.student_api.dto.StudentRequest;
import com.example.student_api.dto.StudentResponse;
import com.example.student_api.entity.Student;
import com.example.student_api.repository.StudentRepository;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.server.ResponseStatusException;

import java.util.List;

@Service
public class StudentService {

    private final StudentRepository studentRepository;

    public StudentService(StudentRepository studentRepository) {
        this.studentRepository = studentRepository;
    }

    @Transactional(readOnly = true)
    public StudentPageResponse getAllStudents(Pageable pageable) {

        Page<Student> studentPage = studentRepository.findAll(pageable);

        List<StudentResponse> students = studentPage.getContent()
                .stream()
                .map(student -> toStudentResponse(student))
                .toList();

        StudentPageResponse response = new StudentPageResponse();

        response.setContent(students);
        response.setPage(studentPage.getNumber());
        response.setSize(studentPage.getSize());
        response.setTotalElements(studentPage.getTotalElements());
        response.setTotalPages(studentPage.getTotalPages());

        return response;
    }

    @Transactional
    public Student createStudent(StudentRequest studentRequest) {

        Student student = new Student();

        student.setName(studentRequest.getName());
        student.setEmail(studentRequest.getEmail());
        student.setAge(studentRequest.getAge());

        return studentRepository.save(student);
    }

    @Transactional(readOnly = true)
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

## service/AuthService.java

```java
package com.example.student_api.service;

import com.example.student_api.dto.LoginRequest;
import com.example.student_api.dto.SignupRequest;
import com.example.student_api.dto.UserResponse;
import com.example.student_api.entity.User;
import com.example.student_api.repository.UserRepository;

import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;

@Service
public class AuthService {

    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;

    public AuthService(
            UserRepository userRepository,
            PasswordEncoder passwordEncoder) {

        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
    }

    public UserResponse signup(SignupRequest signupRequest) {

        User user = new User();

        user.setName(signupRequest.getName());
        user.setEmail(signupRequest.getEmail());

        String hashedPassword =
                passwordEncoder.encode(signupRequest.getPassword());

        user.setPassword(hashedPassword);

        User savedUser = userRepository.save(user);

        UserResponse response = new UserResponse();

        response.setId(savedUser.getId());
        response.setName(savedUser.getName());
        response.setEmail(savedUser.getEmail());

        return response;
    }

    public boolean login(LoginRequest loginRequest) {

        User user = userRepository.findByEmail(loginRequest.getEmail())
                .orElseThrow(() -> new RuntimeException("Invalid email or password"));

        return passwordEncoder.matches(
                loginRequest.getPassword(),
                user.getPassword()
        );
    }
}
```

## controller/AuthController.java

```java
package com.example.student_api.controller;

import com.example.student_api.dto.LoginRequest;
import com.example.student_api.dto.SignupRequest;
import com.example.student_api.dto.UserResponse;
import com.example.student_api.service.AuthService;

import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/auth")
public class AuthController {

    private final AuthService authService;

    public AuthController(AuthService authService) {
        this.authService = authService;
    }

    @PostMapping("/signup")
    public UserResponse signup(@RequestBody SignupRequest signupRequest) {
        return authService.signup(signupRequest);
    }

    @PostMapping("/login")
    public boolean login(@RequestBody LoginRequest loginRequest) {
        return authService.login(loginRequest);
    }
}
```

## controller/StudentController.java

```java
package com.example.student_api.controller;

import com.example.student_api.dto.StudentPageResponse;
import com.example.student_api.dto.StudentRequest;
import com.example.student_api.dto.StudentResponse;
import com.example.student_api.entity.Student;
import com.example.student_api.service.StudentService;
import jakarta.validation.Valid;
import org.springframework.data.domain.Pageable;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/students")
public class StudentController {

    private final StudentService studentService;

    public StudentController(StudentService studentService) {
        this.studentService = studentService;
    }

    @GetMapping
    public StudentPageResponse getAllStudents(Pageable pageable) {

        return studentService.getAllStudents(pageable);
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

## API quick reference

| Operation | Method | Endpoint | Auth | Body | Success |
|-----------|--------|----------|------|------|---------|
| List students (paged, sorted) | <span class="method get">GET</span> | `/students?page=0&size=5&sort=name,asc` | Basic Auth | — | `200` + `StudentPageResponse` |
| Create student | <span class="method post">POST</span> | `/students` | Basic Auth | `StudentRequest` | `201` |
| Read / update / delete | <span class="method get">GET</span> <span class="method put">PUT</span> <span class="method delete">DELETE</span> | `/students/{id}` | — * | — / `StudentRequest` | `200` / `200` / `204` |
| Sign up | <span class="method post">POST</span> | `/auth/signup` | — | `SignupRequest` | `200` + `UserResponse` |
| Log in | <span class="method post">POST</span> | `/auth/login` | — | `LoginRequest` | `200` + `true` / `false` |

\* Not protected yet, because `requestMatchers("/students")` matches only the exact path. Fixed in the next phase.

Example bodies:

```json
// POST /auth/signup
{ "name": "Alice", "email": "alice@example.com", "password": "123456" }

// POST /auth/login
{ "email": "alice@example.com", "password": "123456" }
```

## What's next?

The next phase turns login into a real token system: **JWT access and refresh tokens**, validating the token on every request, protecting all student endpoints, and **USER / ADMIN** roles. 🚀
