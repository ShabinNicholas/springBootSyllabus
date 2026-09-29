# 12. Read APIs (GET)

> Covers **Steps 19–20**: test Get All Students, then build and test Get Student by ID.

## What we'll do

So far we have:

- <span class="method get">GET</span> `/students`: get **all** students
- <span class="method post">POST</span> `/students`: **create** a student

First we'll check that saved students really come back from PostgreSQL. Then we'll add an API to get **one** student by ID.

## Step 19 — Test Get All Students

Keep your Spring Boot app running. In Postman:

- **Method:** `GET`
- **URL:** `http://localhost:8080/students`
- **Body:** none

Click **Send**. You should get something like:

```json
[
  {
    "id": 1,
    "name": "John",
    "email": "john@gmail.com",
    "age": 22
  },
  {
    "id": 2,
    "name": "John",
    "email": "john@gmail.com",
    "age": 22
  }
]
```

?> Your result may have one student or several, depending on how many times you sent the POST request while testing.

## Step 20 — Get Student by ID

We want this:

<span class="method get">GET</span> `/students/2`

to return:

```json
{
  "id": 2,
  "name": "John",
  "email": "john@gmail.com",
  "age": 22
}
```

### 20.1 — The repository (no changes)

`StudentRepository.java` stays as it is:

```java
public interface StudentRepository extends JpaRepository<Student, Long> {
}
```

`JpaRepository` already gives us `findById()`, so there's nothing to add.

### 20.2 — Add a method to the service

In `StudentService.java`, add this below `createStudent()`:

```java
public Student getStudentById(Long id) {
    return studentRepository.findById(id).orElse(null);
}
```

`findById()` returns an `Optional<Student>`, a box that may or may not contain a student. `.orElse(null)` means: *"give me the student if it's there, otherwise give me `null`."* We'll improve this on [page 15](phase-1/15-not-found.md).

### 20.3 — Add the controller endpoint

In `StudentController.java`, add this import:

```java
import org.springframework.web.bind.annotation.PathVariable;
```

Then add this method below `createStudent()`:

```java
@GetMapping("/{id}")
public Student getStudentById(@PathVariable Long id) {
    return studentService.getStudentById(id);
}
```

| Code | Meaning |
|------|---------|
| `@GetMapping("/{id}")` | Handles `GET /students/{id}`, where `{id}` is a placeholder |
| `@PathVariable Long id` | Takes the `{id}` value from the URL (for example `2`) |

Your controller now has three endpoints:

```java
@GetMapping
public List<Student> getAllStudents() {
    return studentService.getAllStudents();
}

@PostMapping
public Student createStudent(@RequestBody Student student) {
    return studentService.createStudent(student);
}

@GetMapping("/{id}")
public Student getStudentById(@PathVariable Long id) {
    return studentService.getStudentById(id);
}
```

### 20.4 — Test Get Student by ID

Restart the app if needed. In Postman:

- **Method:** `GET`
- **URL:** `http://localhost:8080/students/2`

You should get:

```json
{
  "id": 2,
  "name": "John",
  "email": "john@gmail.com",
  "age": 22
}
```

## ✅ Checkpoint

- [ ] `GET /students` returns the students you created
- [ ] `GET /students/{id}` returns a single student

Next: **[13. Update API (PUT)](phase-1/13-update-api.md)** →
