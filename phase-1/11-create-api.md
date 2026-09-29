# 11. Create API (POST)

> Covers **Steps 16–18**: add the POST endpoint, the create logic, and getters/setters on `Student`.

## What we'll do

Add the **Create** operation so we can add a student:

<span class="method post">POST</span> `http://localhost:8080/students`

## Step 16 — Add the POST endpoint to the controller

Open `StudentController.java`. Add these two imports at the top:

```java
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
```

Add this method **inside the class**, below `getAllStudents()`:

```java
@PostMapping
public Student createStudent(@RequestBody Student student) {
    return studentService.createStudent(student);
}
```

| Code | Meaning |
|------|---------|
| `@PostMapping` | This method handles `POST /students` |
| `@RequestBody Student student` | Spring turns the JSON in the request body into a `Student` object |

!> **Don't test it yet.** `createStudent()` doesn't exist in the service yet, so VS Code will show an error on that line for now.

## Step 17 — Add the create logic to the service

Open `StudentService.java` and add this method below `getAllStudents()`:

```java
public Student createStudent(Student student) {
    return studentRepository.save(student);
}
```

Your service now has:

```java
public List<Student> getAllStudents() {
    return studentRepository.findAll();
}

public Student createStudent(Student student) {
    return studentRepository.save(student);
}
```

This connects the POST API to the repository. `save()` inserts the student into PostgreSQL and returns it with its new `id`.

## Step 18 — Add getters and setters to Student

If you test the POST endpoint now, it won't work properly: the saved student comes back empty or without its data.

**Why?** Spring uses a library called **Jackson** to convert JSON into Java objects and back. Our `Student` fields are `private`, and the class has no getters or setters. That means Jackson can't **read** the JSON values into the object, and it can't **write** the object back out as JSON.

This is the point where getters and setters are actually needed. Replace your `Student.java` with:

```java
package com.example.student_api.entity;

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

## Step 18.1 — Test the POST API

1. Save the file.
2. If Spring Boot doesn't restart by itself, stop it (**Ctrl + C**) and run:

   ```powershell
   .\mvnw spring-boot:run
   ```

3. In **Postman**, send:

   - **Method:** `POST`
   - **URL:** `http://localhost:8080/students`
   - **Body → raw → JSON:**

   ```json
   {
     "name": "John",
     "email": "john@gmail.com",
     "age": 22
   }
   ```

You should get the saved student back, **with an `id`**:

```json
{
  "id": 1,
  "name": "John",
  "email": "john@gmail.com",
  "age": 22
}
```

?> Your `id` may be different. It depends on how many students you've already created.

## ✅ Checkpoint

- [ ] The controller has a `@PostMapping` method
- [ ] The service has `createStudent()`
- [ ] `Student` has getters and setters
- [ ] `POST /students` returns the new student with an `id`

Next: **[12. Read APIs (GET)](phase-1/12-read-apis.md)** →
