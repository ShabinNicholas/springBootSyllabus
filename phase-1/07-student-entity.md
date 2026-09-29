# 7. Student Entity

> Covers **Step 9**: create the `Student` entity.

## What we'll do

Now we start building the CRUD API itself. The first thing we need is a **Student entity**.

> An **entity** is a Java class that represents a table in the database.

Our student has four fields:

```text
id
name
email
age
```

Hibernate/JPA uses this class to create the `student` table for us.

## Step 9.1 — Create the entity package

In VS Code, go to `src/main/java/com/example/student_api`. Right-click the main package and create a new folder called **`entity`**.

```text
src
└── main
    └── java
        └── com.example.student_api
            ├── StudentApiApplication.java
            └── entity
```

## Step 9.2 — Create Student.java

Inside `entity`, create **`Student.java`**:

```java
package com.example.student_api.entity;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;

@Entity
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private String email;

    private Integer age;
}
```

?> **Package name:** the first line must match your project. If your `StudentApiApplication.java` starts with `package com.example.student_api;`, then `package com.example.student_api.entity;` is correct. If yours is different, use your own package name.

## What this means

| Code | Meaning |
|------|---------|
| `@Entity` | This class maps to a database table (named `student`) |
| `@Id` | This field is the primary key |
| `@GeneratedValue(strategy = GenerationType.IDENTITY)` | The database generates the ID automatically (1, 2, 3, ...) |
| `private String name;` etc. | Each field becomes a column in the table |

## Don't add getters and setters yet

We'll add getters and setters later, when we can **see why they're needed**. Adding them now just because everyone does wouldn't teach you anything.

## Your structure now

```text
student-api
├── src
│   └── main
│       ├── java
│       │   └── com.example.student_api
│       │       ├── StudentApiApplication.java
│       │       └── entity
│       │           └── Student.java
│       └── resources
│           └── application.properties
├── pom.xml
└── mvnw.cmd
```

!> **Stop here.** Don't create the repository or controller yet.

## ✅ Checkpoint

- [ ] `entity/Student.java` exists with the code above
- [ ] The package name matches your project

Next: **[8. Repository](phase-1/08-repository.md)** →
