# 9. User Entity

> Create a `User` entity and `UserRepository`, and fix the PostgreSQL `user` keyword problem.

## What we'll do

We start replacing Spring Security's temporary in-memory user with **our own users stored in PostgreSQL**. We keep it very simple: no JWT, no login and no roles yet.

## Step 1 — Create User.java

Create `src/main/java/com/example/student_api/entity/User.java`:

```java
package com.example.student_api.entity;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;

@Entity
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private String email;

    private String password;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

| Field | Purpose |
|-------|---------|
| `id` | Unique user ID |
| `name` | The user's name |
| `email` | Used to log in |
| `password` | Will store the **BCrypt hash**, never the plain password |

The database will eventually hold something like:

```text
email:    shabin@gmail.com
password: $2a$10$xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

and **never**:

```text
password: 123456
```

## Step 2 — Create UserRepository

Create `src/main/java/com/example/student_api/repository/UserRepository.java`:

```java
package com.example.student_api.repository;

import com.example.student_api.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Long> {

}
```

Just like `StudentRepository`, this gives us CRUD operations for users:

```text
UserRepository → JpaRepository<User, Long> → CRUD for User → users table
```

## Step 3 — Add findByEmail()

To log in, we need to find a user by their email. Change `UserRepository` to:

```java
package com.example.student_api.repository;

import com.example.student_api.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.Optional;

public interface UserRepository extends JpaRepository<User, Long> {

    Optional<User> findByEmail(String email);

}
```

### What this means

We only wrote the method **name**. Spring Data JPA reads `findByEmail` and generates the query for us: *"find a user whose `email` matches the one I pass in."*

```text
findByEmail("shabin@gmail.com") → users table → matching user → User object
```

It returns `Optional<User>` because the email **might not exist**:

```text
shabin@gmail.com   → user found ✅
unknown@gmail.com  → no user found ❌ (an empty Optional)
```

`Optional` makes us handle the "not found" case on purpose, the same way we did with `findById()` and `orElseThrow()`.

## Step 4 — Restart and check the log

Stop the app (**Ctrl + C**) and run `.\mvnw spring-boot:run`.

The app **starts**, but if you read the log carefully you'll find an error like:

```text
ERROR: syntax error at or near "user"
```

### Why?

In PostgreSQL, **`user` is a reserved keyword**. Hibernate tried to run:

```sql
create table user (...)
```

and PostgreSQL rejected it. So the table **wasn't created**, even though the app is running.

?> **Read the whole log, not just the last line.** `Started StudentApiApplication` doesn't always mean everything worked. Hibernate reports table problems and carries on.

## Step 5 — Name the table "users"

Open `User.java` and add this import:

```java
import jakarta.persistence.Table;
```

Add `@Table(name = "users")` under `@Entity`:

```java
package com.example.student_api.entity;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.Table;

@Entity
@Table(name = "users")
public class User {
```

Now Hibernate creates `users` ✅ instead of `user` ❌. The Java class is still called `User`; only the table name changes.

## Step 6 — Restart and verify

Restart the app. You should **no longer** see `syntax error at or near "user"`, and you should see `Started StudentApiApplication`.

Your database now has:

```text
student_db
├── student
└── users
```

The existing `student` table isn't affected.

?> In psql, `\dt` lists the tables. You should see both `student` and `users`.

## ✅ Checkpoint

- [ ] `entity/User.java` exists with `@Table(name = "users")`
- [ ] `repository/UserRepository.java` has `findByEmail()`
- [ ] The `users` table exists and the log shows no `"user"` syntax error

Next: **[10. Signup with BCrypt](phase-3/10-signup.md)** →
