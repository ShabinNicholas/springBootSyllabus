# 13. Duplicate Emails

> Covers **Step 30.11**: make email unique and return `409 Conflict` for duplicates.

## The problem

Nothing stops two students from having the same email. Send the same POST twice and you get two `kevin@gmail.com` rows. Emails should be **unique**.

We'll protect against duplicates in the **database**, because it's the one place that can't be bypassed. Then we'll turn the database error into a clean API response.

```text
Duplicate email
   ↓
PostgreSQL UNIQUE constraint rejects it
   ↓
DataIntegrityViolationException
   ↓
GlobalExceptionHandler
   ↓
409 Conflict  +  "Email already exists"
```

## Step 30.11.1 — Mark email as unique in the entity

Open `Student.java` and add this import:

```java
import jakarta.persistence.Column;
```

Change the email field to:

```java
@Column(nullable = false, unique = true)
private String email;
```

So the fields are:

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;

private String name;

@Column(nullable = false, unique = true)
private String email;

private Integer age;
```

| Setting | Meaning |
|---------|---------|
| `unique = true` | No two rows may have the same email |
| `nullable = false` | The email column can't be empty (`NULL`) |

Save the file and restart the app.

## Step 30.11.2 — Why the table may not change

`spring.jpa.hibernate.ddl-auto=update` creates **new** tables and columns, but it's careful with **existing** ones. Our `student` table already exists, and it may already contain duplicate emails from testing. A unique rule can't be added while duplicates exist, so Hibernate may skip it. It usually logs a warning instead of stopping the app.

So we'll add the database rule **ourselves** with SQL. First, we have to remove any duplicates.

## Step 30.11.3 — Find duplicate emails

Open psql (or pgAdmin's Query Tool) on `student_db`:

```powershell
psql -h 127.0.0.1 -p 5432 -U postgres -d student_db
```

Find any email that appears more than once:

```sql
SELECT email, COUNT(*)
FROM student
GROUP BY email
HAVING COUNT(*) > 1;
```

If this returns **no rows**, there are no duplicates. Skip to **Step 30.11.5** below.

## Step 30.11.4 — Remove the duplicates

See which rows are duplicated, for example:

```sql
SELECT * FROM student WHERE email = 'kevin@gmail.com' ORDER BY id;
```

Keep one row per email (normally the one with the lowest `id`) and delete the others **by ID**:

```sql
DELETE FROM student WHERE id = 13;
```

!> **Check the IDs before deleting.** `DELETE` can't be undone. Run the `SELECT` first and only delete the IDs you're sure about. This is test data, but it's a good habit for when the data is real.

Run the query from Step 30.11.3 again. It should return no rows.

## Step 30.11.5 — Add the unique constraint

Now add the database rule:

```sql
ALTER TABLE student
ADD CONSTRAINT uk_student_email UNIQUE (email);
```

If it works, PostgreSQL shows:

```text
ALTER TABLE
```

This is the **database-level protection**. Now the entity (`unique = true`) and the database constraint agree.

?> We name the constraint `uk_student_email` (uk = *unique key*) so error messages are easy to read. If you get *"could not create unique index"*, there are still duplicates. Go back to Step 30.11.3.

?> You can check the constraint with `\d student` in psql. Look for `uk_student_email` under *Indexes*. If Hibernate already added its own unique constraint (a long name like `uk_abc123...`), that's okay: having both does no harm.

## Step 30.11.6 — Test a duplicate (see the problem)

Send <span class="method post">POST</span> `http://localhost:8080/students` with an email that already exists:

```json
{
  "name": "Duplicate Kevin",
  "email": "kevin@gmail.com",
  "age": 25
}
```

You get:

```http
500 Internal Server Error
```

That's expected **at this stage**. The database **correctly rejected** the duplicate, but our API doesn't know how to handle that database exception yet, so Spring treats it as an unexpected server error.

## Step 30.11.7 — Find the exception

Look at the **Spring Boot console** right after that request. You'll find something like:

```text
ERROR ... DataIntegrityViolationException ...
ERROR: duplicate key value violates unique constraint "uk_student_email"
  Detail: Key (email)=(kevin@gmail.com) already exists.
```

Two important pieces:

- The exception type: **`DataIntegrityViolationException`**. This is what we'll handle.
- The cause from PostgreSQL: **`duplicate key value violates unique constraint "uk_student_email"`**. The protection works.

?> **Reading the console** like this is how you find out which exception to handle. You'll do it often as a Spring developer.

## Step 30.11.8 — Handle it in GlobalExceptionHandler

Open `GlobalExceptionHandler.java` and add this import:

```java
import org.springframework.dao.DataIntegrityViolationException;
```

Add this method:

```java
@ExceptionHandler(DataIntegrityViolationException.class)
@ResponseStatus(HttpStatus.CONFLICT)
public Map<String, String> handleDataIntegrityViolation(
        DataIntegrityViolationException ex) {

    Map<String, String> error = new HashMap<>();

    error.put("message", "Email already exists");

    return error;
}
```

`409 Conflict` means: *"Your request is valid, but it conflicts with data that already exists."* That's exactly what a duplicate email is.

!> **A simplification to know about:** this handler says *"Email already exists"* for **any** `DataIntegrityViolationException`. Right now the email constraint is the only one that can cause it, so that's fine. If you add more database rules later, you'll need to check which constraint failed before choosing the message.

Save the file and restart the app.

## Step 30.11.9 — Test the duplicate again

Send the same request:

```json
{
  "name": "Duplicate Kevin",
  "email": "kevin@gmail.com",
  "age": 25
}
```

Expected **`409 Conflict`**:

```json
{
  "message": "Email already exists"
}
```

?> This works for **PUT** too: changing a student's email to one that another student already uses also returns `409`.

## 🎉 Phase 2 complete

Your API now protects itself and explains what went wrong:

| Situation | Status | Body |
|-----------|--------|------|
| Created | `201 Created` | `StudentResponse` |
| Read / updated | `200 OK` | `StudentResponse` |
| Deleted | `204 No Content` | — |
| Invalid data | `400 Bad Request` | `{"field": "message", ...}` |
| Student not found | `404 Not Found` | `{"message": "Student not found"}` |
| Duplicate email | `409 Conflict` | `{"message": "Email already exists"}` |

## ✅ Checkpoint

- [ ] `email` has `@Column(nullable = false, unique = true)`
- [ ] The `uk_student_email` constraint exists in PostgreSQL
- [ ] A duplicate email returns `409` with `"Email already exists"`

Next: **[Final Code](phase-2/final-code.md)**: every file in its finished Phase 2 state →
