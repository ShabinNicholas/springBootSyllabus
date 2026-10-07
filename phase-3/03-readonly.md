# 3. Where It Goes, and readOnly

> Where `@Transactional` belongs, method vs class, and `readOnly = true` for reads.

## Step 1 — Put transactions in the service layer

You have:

```java
@Transactional
public Student createStudent(StudentRequest studentRequest) {
```

That's the right place. **The service layer is where transaction boundaries normally go:**

```text
Controller    → handles the HTTP request
    ↓
Service       ← @Transactional (business logic + transaction)
    ↓
Repository    → database access
    ↓
Database
```

Don't change anything.

## Step 2 — On a method vs on the class

On a **method**, only that method is transactional:

```text
StudentService
createStudent()   → @Transactional ✅
updateStudent()   → no @Transactional
deleteStudent()   → no @Transactional
getStudentById()  → no @Transactional
```

On the **class**, every public method gets it by default:

```java
@Service
@Transactional
public class StudentService {
```

```text
StudentService
createStudent()   → @Transactional ✅
updateStudent()   → @Transactional ✅
deleteStudent()   → @Transactional ✅
getStudentById()  → @Transactional ✅
getAllStudents()  → @Transactional ✅
```

**Which should you use?** You don't need to put it on the whole class just because you can. Usually we put `@Transactional` on methods that represent a **business operation that needs a transaction**, especially ones with several database operations.

?> `save()` and `delete()` from `JpaRepository` already run in their own transaction. That's why `updateStudent()` and `deleteStudent()` work without the annotation. `@Transactional` on *your* method matters when you want **several** calls to succeed or fail **together**.

Don't change your code for this step.

## Step 3 — readOnly = true on getAllStudents()

`getAllStudents()` only **reads** data. Spring lets us say so. Change it to:

```java
@Transactional(readOnly = true)
public List<Student> getAllStudents() {
    return studentRepository.findAll();
}
```

This tells Spring: *"this transaction is only for reading."* Hibernate can use it to skip work it would only need for changes, and it makes the method's purpose clear to anyone reading the code.

## Step 4 — readOnly = true on getStudentById()

```java
@Transactional(readOnly = true)
public Student getStudentById(Long id) {
    return studentRepository.findById(id)
            .orElseThrow(() -> new ResponseStatusException(
                    HttpStatus.NOT_FOUND,
                    "Student not found"
            ));
}
```

## Where we are

```text
createStudent()   → @Transactional
getAllStudents()  → @Transactional(readOnly = true)
getStudentById()  → @Transactional(readOnly = true)
```

`updateStudent()` and `deleteStudent()` don't need anything new for now.

!> **`readOnly = true` is not a security feature.** It's mainly a hint to Spring and Hibernate that the method is meant to read data. It doesn't guarantee that the database blocks every possible write.

## ✅ Checkpoint

- [ ] You know why `@Transactional` goes in the service layer
- [ ] `getAllStudents()` and `getStudentById()` use `@Transactional(readOnly = true)`

Next: **[4. Pagination](phase-3/04-pagination.md)** →
