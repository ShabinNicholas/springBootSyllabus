# 17. 404 for Update & Delete

> Covers **Step 25**: return `404 Not Found` from PUT and DELETE when the student doesn't exist.

## The problem

`GET /students/{id}` already returns `404` for a missing student. But:

- **PUT** still uses `.orElse(null)` and `return null;`, so updating a missing student returns `200 OK` with an empty body.
- **DELETE** uses `deleteById()`, which never tells us whether the student existed, so deleting a missing student looks like a success.

Let's fix both.

## Step 25.1 — Update updateStudent()

In `StudentService.java`, replace the current `updateStudent()`:

```java
public Student updateStudent(Long id, Student updatedStudent) {
    Student existingStudent = studentRepository.findById(id).orElse(null);

    if (existingStudent == null) {
        return null;
    }

    existingStudent.setName(updatedStudent.getName());
    existingStudent.setEmail(updatedStudent.getEmail());
    existingStudent.setAge(updatedStudent.getAge());

    return studentRepository.save(existingStudent);
}
```

with:

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

The `null` check is gone. If the student doesn't exist, `orElseThrow` stops the method and Spring returns `404`.

## Step 25.2 — Test update with a missing student

Use an ID that doesn't exist, such as **999**. In Postman:

- **Method:** `PUT`
- **URL:** `http://localhost:8080/students/999`
- **Body → raw → JSON:**

```json
{
  "name": "Test Student",
  "email": "test@gmail.com",
  "age": 20
}
```

You should get **`404 Not Found`**:

```json
{
  "timestamp": "...",
  "status": 404,
  "error": "Not Found",
  "path": "/students/999"
}
```

?> The `"message"` field may not appear, just like in the earlier 404 test. That's okay (see the tip on [page 15](phase-1/15-not-found.md)).

## Step 25.3 — Update deleteStudent()

Replace the current `deleteStudent()`:

```java
public void deleteStudent(Long id) {
    studentRepository.deleteById(id);
}
```

with:

```java
public void deleteStudent(Long id) {
    Student student = studentRepository.findById(id)
            .orElseThrow(() -> new ResponseStatusException(
                    HttpStatus.NOT_FOUND,
                    "Student not found"
            ));

    studentRepository.delete(student);
}
```

### Why are we changing this?

`deleteById(id)` tries to delete straight away, and it doesn't tell us if nothing was there. Instead, we **check first** with `findById(id)`:

- The student exists → delete it → `204 No Content`
- The student doesn't exist → throw → **`404 Not Found`**

!> Don't test it yet. Make the change and save first.

## Step 25.4 — Test delete with a missing student

In Postman:

- **Method:** `DELETE`
- **URL:** `http://localhost:8080/students/999`
- **Body:** none

You should get **`404 Not Found`**:

```json
{
  "timestamp": "...",
  "status": 404,
  "error": "Not Found",
  "path": "/students/999"
}
```

## 🎉 Phase 1 complete

Your Student CRUD API now handles the main success and error cases properly:

| Operation | Existing student | Missing student |
|-----------|------------------|-----------------|
| <span class="method get">GET</span> `/students/{id}` | `200 OK` | `404 Not Found` |
| <span class="method put">PUT</span> `/students/{id}` | `200 OK` | `404 Not Found` |
| <span class="method delete">DELETE</span> `/students/{id}` | `204 No Content` | `404 Not Found` |
| <span class="method post">POST</span> `/students` | `201 Created` | — |
| <span class="method get">GET</span> `/students` | `200 OK` | — |

## ✅ Checkpoint

- [ ] `PUT /students/999` returns `404 Not Found`
- [ ] `DELETE /students/999` returns `404 Not Found`

Next: **[Final Code](phase-1/final-code.md)**: every file in its finished state →
