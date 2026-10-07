# 2. First Validation Rule

> Covers **Steps 26.2–26.5**: make `name` required and turn validation on.

## What we'll do

Validation needs **two pieces**:

1. **Rules** on the class: *"name must not be blank"*
2. **`@Valid`** in the controller: *"check the rules before you run this method"*

If either piece is missing, nothing gets validated.

## Step 26.2 — Add @NotBlank to name

Open `Student.java`. Make sure this import is at the top:

```java
import jakarta.validation.constraints.NotBlank;
```

Find:

```java
private String name;
```

Change it to:

```java
@NotBlank
private String name;
```

So the fields look like:

```java
@NotBlank
private String name;

private String email;
private Integer age;
```

`@NotBlank` means the value **can't be `null`, empty (`""`) or only spaces (`"   "`)**.

Save the file. Don't change anything else yet.

## Step 26.3 — Turn on validation in the controller

Open `StudentController.java` and add this import:

```java
import jakarta.validation.Valid;
```

Find your POST method:

```java
@PostMapping
public ResponseEntity<Student> createStudent(@RequestBody Student student) {
```

Add `@Valid` before `@RequestBody`:

```java
@PostMapping
public ResponseEntity<Student> createStudent(@Valid @RequestBody Student student) {
```

`@Valid` tells Spring: *"Before creating the student, check the validation rules in the `Student` class."*

!> Without `@Valid`, the `@NotBlank` annotation does nothing at all. Spring only checks the rules when you ask it to.

Save the file.

## Step 26.4 — Test an invalid name

Restart the app:

```powershell
.\mvnw spring-boot:run
```

In Postman, send <span class="method post">POST</span> `http://localhost:8080/students` with:

```json
{
  "name": "",
  "email": "test@gmail.com",
  "age": 22
}
```

Because `name` has `@NotBlank`, the request is rejected with:

```http
400 Bad Request
```

The student is **not** saved. Spring stopped the request before your controller method even ran.

## Step 26.5 — Test a valid student

Send the same request with a real name:

```json
{
  "name": "David",
  "email": "david@gmail.com",
  "age": 22
}
```

You should get **`201 Created`** with the new student:

```json
{
  "id": 5,
  "name": "David",
  "email": "david@gmail.com",
  "age": 22
}
```

?> Your `id` will probably be different. It depends on how many students you've created so far.

## ✅ Checkpoint

- [ ] Empty name → `400 Bad Request`
- [ ] Valid name → `201 Created`

Next: **[3. Email & Age Validation](phase-2/03-email-age-validation.md)** →
