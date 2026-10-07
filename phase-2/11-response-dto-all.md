# 11. Response DTO Everywhere

> Covers **Steps 30.7–30.9**: use `StudentResponse` for PUT, GET by ID and GET all.

## Step 30.7 — PUT returns StudentResponse

Your PUT method currently looks like:

```java
@PutMapping("/{id}")
public ResponseEntity<Student> updateStudent(
        @PathVariable Long id,
        @Valid @RequestBody StudentRequest student) {

    Student updatedStudent = studentService.updateStudent(id, student);

    return ResponseEntity.ok(updatedStudent);
}
```

Change the response type and convert the result:

```java
@PutMapping("/{id}")
public ResponseEntity<StudentResponse> updateStudent(
        @PathVariable Long id,
        @Valid @RequestBody StudentRequest student) {

    Student updatedStudent = studentService.updateStudent(id, student);

    StudentResponse response = studentService.toStudentResponse(updatedStudent);

    return ResponseEntity.ok(response);
}
```

### Test

Restart, then use an existing student (for example ID 12). Send <span class="method put">PUT</span> `http://localhost:8080/students/12`:

```json
{
  "name": "Kevin Updated",
  "email": "kevinupdated@gmail.com",
  "age": 27
}
```

Expected **`200 OK`**:

```json
{
  "id": 12,
  "name": "Kevin Updated",
  "email": "kevinupdated@gmail.com",
  "age": 27
}
```

## Step 30.8 — GET by ID returns StudentResponse

Replace:

```java
@GetMapping("/{id}")
public Student getStudentById(@PathVariable Long id) {
    return studentService.getStudentById(id);
}
```

with:

```java
@GetMapping("/{id}")
public StudentResponse getStudentById(@PathVariable Long id) {

    Student student = studentService.getStudentById(id);

    return studentService.toStudentResponse(student);
}
```

### Test

<span class="method get">GET</span> `http://localhost:8080/students/12` should return:

```json
{
  "id": 12,
  "name": "Kevin Updated",
  "email": "kevinupdated@gmail.com",
  "age": 27
}
```

## Step 30.9 — GET all returns a list of StudentResponse

Replace:

```java
@GetMapping
public List<Student> getAllStudents() {
    return studentService.getAllStudents();
}
```

with:

```java
@GetMapping
public List<StudentResponse> getAllStudents() {

    List<Student> students = studentService.getAllStudents();

    return students.stream()
            .map(student -> studentService.toStudentResponse(student))
            .toList();
}
```

Make sure these imports are in the controller:

```java
import java.util.List;
import com.example.student_api.entity.Student;
import com.example.student_api.dto.StudentResponse;
```

### What `stream().map().toList()` does

We have a `List<Student>` but want a `List<StudentResponse>`. The stream converts **each item** in the list:

```text
[Student 3, Student 4, Student 12]
        │  .stream()           → go through the items one by one
        │  .map(toStudentResponse) → convert each Student into a StudentResponse
        │  .toList()           → collect the results into a new list
        ▼
[StudentResponse 3, StudentResponse 4, StudentResponse 12]
```

?> `.toList()` needs Java 16 or later. We're on Java 25, so it works.

### Test

<span class="method get">GET</span> `http://localhost:8080/students` should return something like:

```json
[
  {
    "id": 3,
    "name": "Shabin",
    "email": "shabin@gmail.com",
    "age": 24
  },
  {
    "id": 12,
    "name": "Kevin Updated",
    "email": "kevinupdated@gmail.com",
    "age": 27
  }
]
```

The exact list depends on what's in your database.

## Where we are

Every endpoint that returns student data now uses `StudentResponse`. The `Student` entity never leaves the service and controller layer:

| Endpoint | Accepts | Returns |
|----------|---------|---------|
| <span class="method post">POST</span> `/students` | `StudentRequest` | `StudentResponse` |
| <span class="method get">GET</span> `/students` | — | `List<StudentResponse>` |
| <span class="method get">GET</span> `/students/{id}` | — | `StudentResponse` |
| <span class="method put">PUT</span> `/students/{id}` | `StudentRequest` | `StudentResponse` |
| <span class="method delete">DELETE</span> `/students/{id}` | — | nothing (`204`) |

## ✅ Checkpoint

- [ ] PUT, GET by ID and GET all return `StudentResponse`
- [ ] All three requests still work as before

Next: **[12. Clean 404 Messages](phase-2/12-not-found-handler.md)** →
