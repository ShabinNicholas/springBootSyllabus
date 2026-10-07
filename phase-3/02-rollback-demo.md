# 2. Rollback Demo

> Watch a transaction roll back two saves, then clean up.

## What we'll do

We'll add a **temporary** method that saves two students and then throws an error on purpose. If `@Transactional` works, **neither** student should end up in the database.

!> Everything on this page is temporary. We delete it at the end (Steps 6 and 7).

## Step 1 — Add a test method to the service

In `StudentService.java`, add this method:

```java
@Transactional
public void transactionExample() {

    Student student1 = new Student();
    student1.setName("Transaction Student 1");
    student1.setEmail("transaction1@gmail.com");
    student1.setAge(22);

    studentRepository.save(student1);

    Student student2 = new Student();
    student2.setName("Transaction Student 2");
    student2.setEmail("transaction2@gmail.com");
    student2.setAge(23);

    studentRepository.save(student2);
}
```

## Step 2 — Make it fail on purpose

Add this line at the **end** of `transactionExample()`, after the second `save()`:

```java
throw new RuntimeException("Something went wrong");
```

So both `save()` calls run, and **then** the method fails.

## Step 3 — Add a temporary endpoint

In `StudentController.java`, add:

```java
@PostMapping("/transaction-example")
public ResponseEntity<String> transactionExample() {

    studentService.transactionExample();

    return ResponseEntity.ok("Transaction completed successfully");
}
```

Because the controller already has `@RequestMapping("/students")`, the URL is:

<span class="method post">POST</span> `http://localhost:8080/students/transaction-example`

## Step 4 — Restart

Stop the app (**Ctrl + C**) and run:

```powershell
.\mvnw spring-boot:run
```

Wait for `Started StudentApiApplication`.

## Step 5 — Call it and check the database

1. Send <span class="method post">POST</span> `http://localhost:8080/students/transaction-example` with **no body**.

   You get **`500 Internal Server Error`**. That's expected, because we threw the exception on purpose.

2. Send <span class="method get">GET</span> `http://localhost:8080/students` and look for *Transaction Student 1* and *Transaction Student 2*.

   **Neither is there.** ❌

Both `save()` calls ran before the exception, but `@Transactional` **rolled them back**:

```text
studentRepository.save(student1)
        ↓
studentRepository.save(student2)
        ↓
RuntimeException ❌
        ↓
ROLLBACK
        ↓
Both inserts are undone
```

?> **Want to see the difference?** Temporarily remove `@Transactional` from `transactionExample()`, restart and call it again. Both students *are* saved, even though the request failed. Then put `@Transactional` back (or just delete the method, as below), and remove those two test students with `DELETE /students/{id}`.

?> **Good to know:** by default, Spring rolls back for **unchecked** exceptions (`RuntimeException` and its subclasses) and errors. It does *not* roll back for checked exceptions unless you configure it with `@Transactional(rollbackFor = Exception.class)`.

## Step 6 — Remove the temporary endpoint

In `StudentController.java`, delete **only** this method:

```java
@PostMapping("/transaction-example")
public ResponseEntity<String> transactionExample() {

    studentService.transactionExample();

    return ResponseEntity.ok("Transaction completed successfully");
}
```

## Step 7 — Remove the temporary service method

In `StudentService.java`, delete the **whole** `transactionExample()` method, including the `throw`.

!> **Keep** the `@Transactional` on your real `createStudent()` method.

## ✅ Checkpoint

- [ ] The failing request returned `500` and neither test student was saved
- [ ] `transactionExample()` is removed from the controller and the service
- [ ] `createStudent()` still has `@Transactional`

Next: **[3. Where It Goes, and readOnly](phase-3/03-readonly.md)** →
