# 4. Validate PUT

> Covers **Steps 26.12–26.17**: validate updates too, and learn the difference between `@Email` and `@NotBlank`.

## The problem

We added `@Valid` to POST only. That means someone could **create** a valid student and then **update** it with an age of 5. Validation should apply to every request that sends student data.

## Step 26.12 — Add @Valid to PUT

Open `StudentController.java`. Find:

```java
@PutMapping("/{id}")
public ResponseEntity<Student> updateStudent(
        @PathVariable Long id,
        @RequestBody Student student) {
```

Change only `@RequestBody Student student` to `@Valid @RequestBody Student student`:

```java
@PutMapping("/{id}")
public ResponseEntity<Student> updateStudent(
        @PathVariable Long id,
        @Valid @RequestBody Student student) {
```

Save the file.

## Step 26.13 — Test an invalid update

Use Michael's ID from the previous page (7 in this example). Send <span class="method put">PUT</span> `http://localhost:8080/students/7`:

```json
{
  "name": "Michael Updated",
  "email": "michaelupdated@gmail.com",
  "age": 15
}
```

Expected: **`400 Bad Request`**, because the age must be at least 18.

## Step 26.14 — Test a valid update

```json
{
  "name": "Michael Updated",
  "email": "michaelupdated@gmail.com",
  "age": 25
}
```

Expected: **`200 OK`** with the updated student.

Both endpoints now use `@Valid`:

```java
@PostMapping
public ResponseEntity<Student> createStudent(@Valid @RequestBody Student student)

@PutMapping("/{id}")
public ResponseEntity<Student> updateStudent(
        @PathVariable Long id,
        @Valid @RequestBody Student student)
```

## Step 26.15 — A surprising test: empty email

Send <span class="method post">POST</span> `http://localhost:8080/students`:

```json
{
  "name": "No Email",
  "email": "",
  "age": 25
}
```

This **succeeds** with `201 Created`. That's expected, and it teaches something important.

**`@Email` checks the *format* of a value, but it does not require a value.** An empty string has no format to break, so it passes:

| Value | `@Email` result |
|-------|-----------------|
| `""` | ✅ allowed |
| `"abc"` | ❌ rejected |
| `"user@gmail.com"` | ✅ allowed |

To make the email **required** *and* **valid**, we need both annotations.

## Step 26.16 — Make email required

In `Student.java`, change:

```java
@Email
private String email;
```

to:

```java
@NotBlank
@Email
private String email;
```

Your fields are now:

```java
@NotBlank
private String name;

@NotBlank
@Email
private String email;

@Min(18)
private Integer age;
```

Save the file.

## Step 26.17 — Test the required email

Restart if needed, then send the same request again:

```json
{
  "name": "No Email",
  "email": "",
  "age": 25
}
```

This time: **`400 Bad Request`**. `@NotBlank` catches the empty value, and `@Email` checks the format of anything that isn't empty.

?> You probably have a "No Email" student saved from Step 26.15. You can delete it with <span class="method delete">DELETE</span> `/students/{id}` if you like.

## Where we are

Validation works on **POST and PUT**. But the response is still just a generic `400 Bad Request`. The client can't tell **which** field was wrong. We fix that next.

## ✅ Checkpoint

- [ ] PUT with age 15 → `400`, PUT with age 25 → `200`
- [ ] POST with an empty email → `400`

Next: **[5. Custom Messages](phase-2/05-custom-messages.md)** →
