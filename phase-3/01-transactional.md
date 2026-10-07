# 1. @Transactional

> Add `@Transactional` to `createStudent()` and understand what it does.

## What is a transaction?

Sometimes one business action needs **several** database operations. For example:

```text
Operation 1 → Create a student
Operation 2 → Create another record
Operation 3 → Update something
```

What if Operation 3 fails? Without a transaction, Operations 1 and 2 are already saved, and the data is half-finished. With a transaction:

```text
If Operation 3 fails
        ↓
Roll back Operations 1 and 2
```

Either **everything succeeds**, or **nothing is saved**. That's a transaction: one *unit of work*.

## Step 1 — Add @Transactional to createStudent()

Open `StudentService.java` and add this import:

```java
import org.springframework.transaction.annotation.Transactional;
```

Add `@Transactional` directly above `createStudent()`:

```java
@Transactional
public Student createStudent(StudentRequest studentRequest) {
```

Don't change anything else.

## Step 2 — What @Transactional does

Your method now looks like this:

```java
@Transactional
public Student createStudent(StudentRequest studentRequest) {

    Student student = new Student();

    student.setName(studentRequest.getName());
    student.setEmail(studentRequest.getEmail());
    student.setAge(studentRequest.getAge());

    return studentRepository.save(student);
}
```

When Spring sees `@Transactional`, it **starts a database transaction before the method runs**, and finishes it when the method ends:

```text
createStudent() starts
        ↓
Transaction starts
        ↓
studentRepository.save()
        ↓
Did everything succeed?
     ↙          ↘
   YES           NO (exception)
    ↓             ↓
  COMMIT       ROLLBACK
```

This method has only **one** database operation, so you won't notice a difference yet. The benefit shows up when a method does **several** operations, which we'll see on the next page.

!> **`@Transactional` does not mean "save this to the database".** It means: *"treat all the database operations inside this method as one transaction."* `save()` still does the saving.

## ✅ Checkpoint

- [ ] `StudentService` imports `org.springframework.transaction.annotation.Transactional`
- [ ] `createStudent()` has `@Transactional`
- [ ] You can explain commit vs rollback in your own words

Next: **[2. Rollback Demo](phase-3/02-rollback-demo.md)** →
