# 4. Pagination

> Return students one page at a time with `Pageable`.

## Why pagination?

`GET /students` returns **every** student. That's fine for 10 students, but:

```text
10 students         → fine
1,000 students      → large response
100,000 students    → problematic
1,000,000 students  → definitely not something we want to return at once
```

**Pagination** returns one chunk at a time:

<span class="method get">GET</span> `/students?page=0&size=5`

```text
Page 0 → first 5 students
Page 1 → next 5 students
Page 2 → next 5 students
```

?> Pages start at **0**, not 1.

## Step 1 — Check the repository (no changes)

Open `StudentRepository.java`. It should be:

```java
package com.example.student_api.repository;

import com.example.student_api.entity.Student;
import org.springframework.data.jpa.repository.JpaRepository;

public interface StudentRepository extends JpaRepository<Student, Long> {
}
```

**You don't need to add anything.** `JpaRepository` already has `findAll(Pageable pageable)`.

## Step 2 — Change getAllStudents() in the service

In `StudentService.java`, replace:

```java
@Transactional(readOnly = true)
public List<Student> getAllStudents() {
    return studentRepository.findAll();
}
```

with:

```java
@Transactional(readOnly = true)
public Page<Student> getAllStudents(Pageable pageable) {
    return studentRepository.findAll(pageable);
}
```

Add these imports:

```java
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
```

What changed:

```text
Before:  findAll()          → List<Student>  → all students
After:   findAll(pageable)  → Page<Student>  → only the requested page
```

!> The controller now shows an error, because it still calls the old `getAllStudents()` with no arguments. That's expected. We fix it next.

## Step 3 — Accept pagination in the controller

In `StudentController.java`, replace:

```java
@GetMapping
public List<StudentResponse> getAllStudents() {

    List<Student> students = studentService.getAllStudents();

    return students.stream()
            .map(student -> studentService.toStudentResponse(student))
            .toList();
}
```

with:

```java
@GetMapping
public List<StudentResponse> getAllStudents(Pageable pageable) {

    Page<Student> students = studentService.getAllStudents(pageable);

    return students.getContent().stream()
            .map(student -> studentService.toStudentResponse(student))
            .toList();
}
```

Add these imports:

```java
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
```

### What Pageable does

Spring reads query parameters like `?page=0&size=5` and **builds the `Pageable` object for you**:

```text
GET /students?page=0&size=5   →   page = 0, size = 5
```

`students.getContent()` gives us just the list of students on that page. On the next page we'll also return page information (total pages, total elements and so on).

?> If you don't send `page` or `size`, Spring uses **page 0** and a default **size of 20**.

## Step 4 — Restart

Stop the app (**Ctrl + C**) and run `.\mvnw spring-boot:run`. Wait for `Started StudentApiApplication`.

## Step 5 — Test pagination

<span class="method get">GET</span> `http://localhost:8080/students?page=0&size=5` returns **only 5 students**.

<span class="method get">GET</span> `http://localhost:8080/students?page=1&size=5` returns **the next 5**.

```text
page=0, size=5  →  students 1–5   (for example IDs 3, 4, 5, 6, 7)
page=1, size=5  →  students 6–10  (for example IDs 8, 9, 10, 11, 12)
```

Spring Data JPA works out which rows belong on each page. We didn't write any SQL.

?> You need more than 5 students to see a difference. If you have fewer, create a few more with `POST /students`, or try `size=2`.

## One limitation

The response is still a plain array:

```json
[
  { "id": 3, "name": "Shabin", "email": "shabin@gmail.com", "age": 24 }
]
```

A frontend usually also needs to know the current page, the page size, the total number of students, the total number of pages, and whether there's a next or previous page. That's next.

## ✅ Checkpoint

- [ ] The service returns `Page<Student>` from `findAll(pageable)`
- [ ] The controller takes a `Pageable` parameter
- [ ] `page=0&size=5` and `page=1&size=5` return different students

Next: **[5. Page Response DTO](phase-3/05-page-response.md)** →
