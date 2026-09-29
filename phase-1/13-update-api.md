# 13. Update API (PUT)

> Covers **Step 21**: add the Update operation.

## What we'll do

Add the **Update** operation:

<span class="method put">PUT</span> `/students/{id}`

For example, to update student 2 we send:

<span class="method put">PUT</span> `http://localhost:8080/students/2`

```json
{
  "name": "John Updated",
  "email": "johnupdated@gmail.com",
  "age": 23
}
```

## Step 21.1 — Add the update method to the service

In `StudentService.java`, add this below `getStudentById()`:

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

### What this does

1. **Find** the existing student in the database:

   ```java
   Student existingStudent = studentRepository.findById(id).orElse(null);
   ```

2. **Stop** if the student doesn't exist:

   ```java
   if (existingStudent == null) {
       return null;
   }
   ```

3. **Copy** the new values onto the existing student:

   ```java
   existingStudent.setName(updatedStudent.getName());
   existingStudent.setEmail(updatedStudent.getEmail());
   existingStudent.setAge(updatedStudent.getAge());
   ```

4. **Save** it. Because `existingStudent` already has an `id`, `save()` runs an `UPDATE` instead of an `INSERT`:

   ```java
   return studentRepository.save(existingStudent);
   ```

!> Don't change the controller yet. Add only this method and save.

## Step 21.2 — Add the PUT endpoint to the controller

In `StudentController.java`, add this import:

```java
import org.springframework.web.bind.annotation.PutMapping;
```

Add this method below `getStudentById()`:

```java
@PutMapping("/{id}")
public Student updateStudent(
        @PathVariable Long id,
        @RequestBody Student student) {
    return studentService.updateStudent(id, student);
}
```

This method uses both:

- `@PathVariable`: **which** student to update (from the URL)
- `@RequestBody`: the **new values** (from the JSON body)

Your controller now has:

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
```

## Step 21.3 — Test the PUT API

Restart the app if needed. In Postman:

- **Method:** `PUT`
- **URL:** `http://localhost:8080/students/2`
- **Body → raw → JSON:**

```json
{
  "name": "John Updated",
  "email": "johnupdated@gmail.com",
  "age": 23
}
```

You should get:

```json
{
  "id": 2,
  "name": "John Updated",
  "email": "johnupdated@gmail.com",
  "age": 23
}
```

## ✅ Checkpoint

- [ ] The service has `updateStudent()`
- [ ] The controller has a `@PutMapping("/{id}")` method
- [ ] `PUT /students/{id}` returns the updated student

Next: **[14. Delete API (DELETE)](phase-1/14-delete-api.md)** →
