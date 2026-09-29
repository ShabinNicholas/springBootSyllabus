# 8. Repository

> Covers **Steps 10–11**: create the `StudentRepository` and check that Spring picks it up.

## What we'll do

Our application needs a way to talk to the `student` table. That's the job of a **repository**.

Spring Data JPA gives us the common database operations for free, so we don't need to write SQL for basic CRUD.

## Step 10.1 — Create the repository package

Inside `src/main/java/com/example/student_api`, create a new folder called **`repository`**.

```text
com.example.student_api
├── StudentApiApplication.java
├── entity
│   └── Student.java
└── repository
```

## Step 10.2 — Create StudentRepository.java

Inside `repository`, create **`StudentRepository.java`**:

```java
package com.example.student_api.repository;

import com.example.student_api.entity.Student;
import org.springframework.data.jpa.repository.JpaRepository;

public interface StudentRepository extends JpaRepository<Student, Long> {
}
```

That's it. 😄

## What this means

Notice that this is an **interface**, not a class. We never write the implementation. Spring creates it for us when the app starts.

`JpaRepository<Student, Long>` means:

- `Student` is the entity this repository works with
- `Long` is the type of the entity's `id`

Because we extend `JpaRepository`, we get methods such as:

| Method | What it does |
|--------|--------------|
| `save()` | Insert a new row, or update an existing one |
| `findAll()` | Get all rows |
| `findById()` | Get one row by ID (returns an `Optional`) |
| `deleteById()` | Delete a row by ID |
| `existsById()` | Check whether a row exists |

We'll use these when we build the CRUD API.

## Step 11 — Verify the entity and repository

Stop the app if it's running (**Ctrl + C**), then start it again:

```powershell
.\mvnw spring-boot:run
```

Look for lines like:

```text
Finished Spring Data repository scanning ... Found 1 JPA repository interface.
...
Started StudentApiApplication
```

You should also see some Hibernate SQL, such as a `create table student (...)` statement.

## Step 11.1 — Check PostgreSQL

Open **pgAdmin** and expand:

```text
student_db
└── Schemas
    └── public
        └── Tables
            └── student
```

You should find a **`student`** table. Hibernate created it automatically because of:

- `@Entity` on the `Student` class, and
- `spring.jpa.hibernate.ddl-auto=update` in `application.properties`

?> No pgAdmin? You can check in psql instead: connect with `\c student_db`, then run `\dt` to list tables.

!> **Stop here.** Don't create the controller yet.

## ✅ Checkpoint

- [ ] The app starts and reports `Found 1 JPA repository interface`
- [ ] The `student` table exists in `student_db`

Next: **[9. Service Layer](phase-1/09-service-layer.md)** →
