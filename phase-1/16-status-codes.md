# 16. HTTP Status Codes

> Covers **Step 24**: return status codes that describe what happened.

## Why status codes matter

Our APIs work, but they return the default `200 OK` for almost everything. In a REST API, the **status code** tells the client what happened:

| Status | Meaning | Use it for |
|--------|---------|------------|
| `200 OK` | Success, here's the data | GET and PUT |
| `201 Created` | A new resource was created | POST |
| `204 No Content` | Success, nothing to send back | DELETE |
| `404 Not Found` | That resource doesn't exist | Missing student |

To control the status code, we return **`ResponseEntity`** from the controller instead of a plain object.

## Step 24.1 — POST returns 201 Created

Open `StudentController.java` and add this import:

```java
import org.springframework.http.ResponseEntity;
```

Replace:

```java
@PostMapping
public Student createStudent(@RequestBody Student student) {
    return studentService.createStudent(student);
}
```

with:

```java
@PostMapping
public ResponseEntity<Student> createStudent(@RequestBody Student student) {
    Student createdStudent = studentService.createStudent(student);

    return ResponseEntity.status(201).body(createdStudent);
}
```

### What's happening here?

Instead of returning just a `Student`, we return a `ResponseEntity<Student>`. That lets us control the whole HTTP response.

| Code | Meaning |
|------|---------|
| `ResponseEntity.status(201)` | Set the HTTP status to `201 Created` |
| `.body(createdStudent)` | Put the new student in the response body |

?> `ResponseEntity.status(HttpStatus.CREATED)` does the same thing and is easier to read, if you prefer names over numbers.

## Step 24.2 — Test the POST API

In Postman:

- **Method:** `POST`
- **URL:** `http://localhost:8080/students`
- **Body → raw → JSON:**

```json
{
  "name": "Alice",
  "email": "alice@gmail.com",
  "age": 21
}
```

The status should now be **`201 Created`**, with the new student in the body:

```json
{
  "id": 4,
  "name": "Alice",
  "email": "alice@gmail.com",
  "age": 21
}
```

?> Your `id` may be different depending on your database.

## Step 24.3 — PUT returns 200 OK

Replace the current `updateStudent()` in the controller:

```java
@PutMapping("/{id}")
public Student updateStudent(
        @PathVariable Long id,
        @RequestBody Student student) {
    return studentService.updateStudent(id, student);
}
```

with:

```java
@PutMapping("/{id}")
public ResponseEntity<Student> updateStudent(
        @PathVariable Long id,
        @RequestBody Student student) {

    Student updatedStudent = studentService.updateStudent(id, student);

    return ResponseEntity.ok(updatedStudent);
}
```

You already added the `ResponseEntity` import in Step 24.1.

`ResponseEntity.ok(updatedStudent)` means: *return the updated student with status **`200 OK`**.*

## Step 24.4 — Test the PUT API

Use one of your existing students, for example Alice with ID 4. In Postman:

- **Method:** `PUT`
- **URL:** `http://localhost:8080/students/4`
- **Body → raw → JSON:**

```json
{
  "name": "Alice Updated",
  "email": "aliceupdated@gmail.com",
  "age": 22
}
```

Expected: **`200 OK`** with:

```json
{
  "id": 4,
  "name": "Alice Updated",
  "email": "aliceupdated@gmail.com",
  "age": 22
}
```

?> The fields may appear in a different order in your response. That's normal.

## Step 24.5 — DELETE returns 204 No Content

Our DELETE method returns `void`, so Spring sends `200 OK`. Since there's no response body, **`204 No Content`** is a better fit.

Replace:

```java
@DeleteMapping("/{id}")
public void deleteStudent(@PathVariable Long id) {
    studentService.deleteStudent(id);
}
```

with:

```java
@DeleteMapping("/{id}")
public ResponseEntity<Void> deleteStudent(@PathVariable Long id) {
    studentService.deleteStudent(id);

    return ResponseEntity.noContent().build();
}
```

`ResponseEntity.noContent()` means: *the operation worked, but there's no response body.* `.build()` creates the response. `Void` (capital V) means the response has no body type.

## Step 24.6 — Test the DELETE API

Delete Alice (ID 4). In Postman:

- **Method:** `DELETE`
- **URL:** `http://localhost:8080/students/4`
- **Body:** none

You should now get **`204 No Content`** with no response body. That's exactly what we want.

?> Then run `GET /students` to confirm she's gone.

## Where we are now

| Operation | Method | Endpoint | Status |
|-----------|--------|----------|--------|
| Create | <span class="method post">POST</span> | `/students` | ✅ `201 Created` |
| Read all | <span class="method get">GET</span> | `/students` | ✅ `200 OK` |
| Read one | <span class="method get">GET</span> | `/students/{id}` | ✅ `200 OK` |
| Update | <span class="method put">PUT</span> | `/students/{id}` | ✅ `200 OK` |
| Delete | <span class="method delete">DELETE</span> | `/students/{id}` | ✅ `204 No Content` |
| Student not found | <span class="method get">GET</span> | `/students/{id}` | ✅ `404 Not Found` |

## ✅ Checkpoint

- [ ] POST returns `201 Created`
- [ ] PUT returns `200 OK`
- [ ] DELETE returns `204 No Content`

Next: **[17. 404 for Update & Delete](phase-1/17-not-found-update-delete.md)** →
