# 14. Delete API (DELETE)

> Covers **Step 22**: add the final basic CRUD operation.

## What we'll do

Add the **Delete** operation:

<span class="method delete">DELETE</span> `/students/{id}`

For example, `DELETE http://localhost:8080/students/2` deletes the student with ID 2.

## Step 22.1 — Add the delete method to the service

In `StudentService.java`, add this below `updateStudent()`:

```java
public void deleteStudent(Long id) {
    studentRepository.deleteById(id);
}
```

That's all the service needs. `JpaRepository` already gives us `deleteById()`, which deletes the row with that ID.

!> Don't change the controller yet. Add only this method and save.

## Step 22.2 — Add the DELETE endpoint to the controller

In `StudentController.java`, add this import:

```java
import org.springframework.web.bind.annotation.DeleteMapping;
```

Add this method below `updateStudent()`:

```java
@DeleteMapping("/{id}")
public void deleteStudent(@PathVariable Long id) {
    studentService.deleteStudent(id);
}
```

Your controller now has all the CRUD endpoints:

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

@PutMapping("/{id}")
public Student updateStudent(
        @PathVariable Long id,
        @RequestBody Student student) {
    return studentService.updateStudent(id, student);
}

@DeleteMapping("/{id}")
public void deleteStudent(@PathVariable Long id) {
    studentService.deleteStudent(id);
}
```

## Step 22.3 — Test the DELETE API

We'll delete the student we just updated (ID 2). Restart the app if needed. In Postman:

- **Method:** `DELETE`
- **URL:** `http://localhost:8080/students/2`
- **Body:** none

Click **Send**. You'll probably see:

```http
200 OK
```

with an **empty response body**. That's normal. Our controller method returns `void`, so there's nothing to send back.

?> Run `GET /students` afterwards to confirm the student is gone.

## 🎉 Basic CRUD done

You now have all five operations working:

| Operation | Method | Endpoint |
|-----------|--------|----------|
| Create | <span class="method post">POST</span> | `/students` |
| Read all | <span class="method get">GET</span> | `/students` |
| Read one | <span class="method get">GET</span> | `/students/{id}` |
| Update | <span class="method put">PUT</span> | `/students/{id}` |
| Delete | <span class="method delete">DELETE</span> | `/students/{id}` |

The next pages make the API behave more **correctly**: proper errors for missing students, and proper HTTP status codes.

## ✅ Checkpoint

- [ ] The service has `deleteStudent()`
- [ ] The controller has a `@DeleteMapping("/{id}")` method
- [ ] `DELETE /students/{id}` returns `200 OK` and the student is gone

Next: **[15. 404 Not Found](phase-1/15-not-found.md)** →
