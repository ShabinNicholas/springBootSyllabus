# 9. DTO for PUT

> Covers **Steps 29.9–29.15**: remove validation from the entity and use `StudentRequest` for PUT.

## Step 29.9 — Remove the validation imports from Student

Now that `StudentRequest` handles validation for POST, the entity doesn't need it.

!> **Do this page in order.** PUT still uses the `Student` entity until Step 29.12. Between those steps, PUT requests won't be validated. That's fine, because we fix it in a few minutes.

Open `Student.java` and remove these three imports:

```java
import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.Min;
import jakarta.validation.constraints.NotBlank;
```

The annotations on the fields will turn red. That's expected.

## Step 29.10 — Remove the validation annotations

Remove these annotations from the fields:

```java
@NotBlank(message = "Name is required")
@NotBlank(message = "Email is required")
@Email(message = "Email must be valid")
@Min(value = 18, message = "Age must be at least 18")
```

So the fields are back to:

```java
private String name;
private String email;
private Integer age;
```

`Student` is now **purely a database entity**, and `StudentRequest` handles validation of what the API accepts:

```text
StudentRequest
├── Validation
│   ├── Name required
│   ├── Email required
│   ├── Email format
│   └── Age validation
│
Student
└── Database entity
```

## Step 29.11 — Re-test POST validation

Restart the app and send an invalid POST:

```json
{
  "name": "",
  "email": "test@gmail.com",
  "age": 25
}
```

You should still get `400` with `{"name": "Name is required"}`. This proves the rules now come **only from the DTO**, because the entity has none left.

## Step 29.12 — Use StudentRequest in PUT

Open `StudentController.java`. Find:

```java
@PutMapping("/{id}")
public ResponseEntity<Student> updateStudent(
        @PathVariable Long id,
        @Valid @RequestBody Student student) {
```

Change only `Student student` to `StudentRequest student`:

```java
@PutMapping("/{id}")
public ResponseEntity<Student> updateStudent(
        @PathVariable Long id,
        @Valid @RequestBody StudentRequest student) {
```

!> **You'll get a red error** on the service call. That's expected. We fix the service next.

## Step 29.13 — Change updateStudent() in the service

In `StudentService.java`, replace:

```java
public Student updateStudent(Long id, Student updatedStudent) {
    Student existingStudent = studentRepository.findById(id)
            .orElseThrow(() -> new ResponseStatusException(
                    HttpStatus.NOT_FOUND,
                    "Student not found"
            ));

    existingStudent.setName(updatedStudent.getName());
    existingStudent.setEmail(updatedStudent.getEmail());
    existingStudent.setAge(updatedStudent.getAge());

    return studentRepository.save(existingStudent);
}
```

with:

```java
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
```

The only difference: the service now receives a `StudentRequest` and copies its values into the existing entity.

?> You already added `import com.example.student_api.dto.StudentRequest;` on the previous page, so don't add it again.

## Step 29.14 — Test PUT with the DTO

Restart the app. Use Sarah's ID from the previous page (10 in this example). Send <span class="method put">PUT</span> `http://localhost:8080/students/10`:

```json
{
  "name": "Sarah Updated",
  "email": "sarahupdated@gmail.com",
  "age": 24
}
```

Expected **`200 OK`**:

```json
{
  "id": 10,
  "name": "Sarah Updated",
  "email": "sarahupdated@gmail.com",
  "age": 24
}
```

## Step 29.15 — Test PUT validation through the DTO

```json
{
  "name": "Sarah Updated",
  "email": "invalid-email",
  "age": 24
}
```

Expected **`400 Bad Request`**:

```json
{
  "email": "Email must be valid"
}
```

POST and PUT now both use `StudentRequest` validation.

## What you learned

```text
                    ┌─────────────────┐
POST / PUT  ──────→ │ StudentRequest  │
                    │  + Validation   │
                    └────────┬────────┘
                             ↓
                       StudentService
                             ↓
                    ┌─────────────────┐
                    │ Student Entity  │
                    └────────┬────────┘
                             ↓
                          Database
```

| Class | Role |
|-------|------|
| **Entity** | Represents your database table |
| **DTO** | Represents the data your API accepts |
| **Validation** | Belongs on the DTO, for incoming request data |

## ✅ Checkpoint

- [ ] `Student.java` has no validation annotations or imports
- [ ] PUT accepts `StudentRequest`
- [ ] Valid PUT → `200`, invalid PUT → `400` with message

Next: **[10. Response DTO](phase-2/10-response-dto.md)** →
