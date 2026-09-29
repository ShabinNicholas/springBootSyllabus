# 6. Connect to PostgreSQL

> Covers **Steps 7–8**: configure Spring Boot to use `student_db` and check the connection.

## What we'll do

We'll tell Spring Boot: *"Use my PostgreSQL `student_db` database."* We do this in `application.properties`.

```text
src
└── main
    └── resources
        └── application.properties
```

## Step 7.1 — Open application.properties

In VS Code, open `src/main/resources/application.properties`. It may be almost empty.

## Step 7.2 — Add the configuration

Replace everything in the file with:

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/student_db
spring.datasource.username=postgres
spring.datasource.password=YOUR_POSTGRES_PASSWORD

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

!> Replace `YOUR_POSTGRES_PASSWORD` with your real `postgres` password. **Don't put quotes around it.** For example, if your password were `MyPassword123`, the line would be `spring.datasource.password=MyPassword123`.

?> **Keep your password out of GitHub.** If you push this project to a public repository, anyone can read `application.properties`. Use a throwaway local password, or don't commit this file with a real password in it.

## What these settings mean

| Setting | Meaning |
|---------|---------|
| `spring.datasource.url` | Where the PostgreSQL database is |
| `spring.datasource.username` | PostgreSQL username |
| `spring.datasource.password` | PostgreSQL password |
| `spring.jpa.hibernate.ddl-auto=update` | Hibernate creates and updates tables to match our Java entities |
| `spring.jpa.show-sql=true` | Prints the SQL that Hibernate runs to the terminal |
| `spring.jpa.properties.hibernate.format_sql=true` | Formats that SQL so it's easier to read |

Save the file with **Ctrl + S**.

## Step 8 — Verify the connection

If the app is still running from before, stop it with **Ctrl + C**. Then run it again from the project folder:

```powershell
.\mvnw spring-boot:run
```

Look for:

```text
Tomcat initialized with port 8080
...
Started StudentApiApplication
```

You may also see messages from Hibernate, because JPA is now connected.

You should **not** see:

```text
Failed to configure a DataSource
```

or:

```text
Failed to determine a suitable driver class
```

?> If you get an error, **don't change anything at random**. Read the error section first. Common causes are a wrong password, PostgreSQL not running, or a typo in the database name.

## ✅ Checkpoint

- [ ] `application.properties` has your database settings
- [ ] The app starts with `Started StudentApiApplication` and no DataSource errors

Next: **[7. Student Entity](phase-1/07-student-entity.md)** →
