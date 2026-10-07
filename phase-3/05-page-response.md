# 5. Page Response DTO

> Return the students **plus** page information in a `StudentPageResponse`.

## What we'll build

Instead of a plain array, `GET /students` will return:

```json
{
  "content": [ ... ],
  "page": 0,
  "size": 5,
  "totalElements": 10,
  "totalPages": 2
}
```

## Step 1 — Create StudentPageResponse

Inside `src/main/java/com/example/student_api/dto`, create **`StudentPageResponse.java`**:

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

| Field | Meaning |
|-------|---------|
| `content` | The students on this page (as `StudentResponse`) |
| `page` | The current page number (starting at 0) |
| `size` | How many students per page |
| `totalElements` | The total number of students in the database |
| `totalPages` | How many pages there are in total |

!> Only create the class for now. Don't change the service or controller yet.

## Step 2 — Build it in the service

In `StudentService.java`, add this import:

```java
import com.example.student_api.dto.StudentPageResponse;
```

Replace:

```java
@Transactional(readOnly = true)
public Page<Student> getAllStudents(Pageable pageable) {
    return studentRepository.findAll(pageable);
}
```

with:

```java
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
```

?> You already have the `List`, `Page`, `Pageable` and `StudentResponse` imports from earlier steps.

What's happening:

```text
Database
   ↓
Page<Student>
   ↓
Convert each Student → StudentResponse
   ↓
Add students + page information
   ↓
StudentPageResponse
```

!> Don't change the controller yet. It shows an error until the next step.

## Step 3 — Return it from the controller

In `StudentController.java`, add this import:

```java
import com.example.student_api.dto.StudentPageResponse;
```

Replace:

```java
@GetMapping
public List<StudentResponse> getAllStudents(Pageable pageable) {

    Page<Student> students = studentService.getAllStudents(pageable);

    return students.getContent().stream()
            .map(student -> studentService.toStudentResponse(student))
            .toList();
}
```

with:

```java
@GetMapping
public StudentPageResponse getAllStudents(Pageable pageable) {

    return studentService.getAllStudents(pageable);
}
```

The controller is now much simpler, because the conversion happens in the service.

You can now remove these imports from the controller if nothing else uses them:

```java
import java.util.List;
import org.springframework.data.domain.Page;
```

**Keep** `import org.springframework.data.domain.Pageable;`, because the controller still uses `Pageable`.

## Step 4 — Restart

Stop the app (**Ctrl + C**) and run `.\mvnw spring-boot:run`.

## Step 5 — Test

<span class="method get">GET</span> `http://localhost:8080/students?page=0&size=5`

```json
{
  "content": [
    {
      "id": 3,
      "name": "Shabin",
      "email": "shabin@gmail.com",
      "age": 24
    }
  ],
  "page": 0,
  "size": 5,
  "totalElements": 10,
  "totalPages": 2
}
```

`content` holds your first 5 students. Then try <span class="method get">GET</span> `http://localhost:8080/students?page=1&size=5`:

```json
{
  "content": [ ... ],
  "page": 1,
  "size": 5,
  "totalElements": 10,
  "totalPages": 2
}
```

?> Your `totalElements` and `totalPages` depend on how many students are in your database.

Now a frontend can build pagination buttons like `← Previous  1  2  Next →`, and it knows whether another page exists without guessing.

## ✅ Checkpoint

- [ ] `dto/StudentPageResponse.java` exists
- [ ] The service returns `StudentPageResponse`
- [ ] `GET /students?page=0&size=5` returns `content` plus page information

Next: **[6. Sorting](phase-3/06-sorting.md)** →
