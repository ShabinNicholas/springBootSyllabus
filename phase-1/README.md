# Phase 1 — Student CRUD API

In this phase we build a simple **Student Management API** with Spring Boot. It has **no signup, login, JWT or roles** yet. We will add authentication in a later phase, once the CRUD API works properly.

## What we'll build

A REST API that can:

- **Create** a student
- **Get all** students
- **Get** a student **by ID**
- **Update** a student
- **Delete** a student

Each student has an `id`, `name`, `email` and `age`.

## Tech stack

| Tool | What it does |
|------|--------------|
| **Java** | The programming language |
| **Spring Boot** | Sets up and runs the application |
| **Spring Web** | Builds the REST endpoints and runs the embedded Tomcat server |
| **Spring Data JPA** | Talks to the database without us writing SQL |
| **PostgreSQL** | The database that stores our students |
| **Maven** (via `mvnw`) | Downloads dependencies and builds and runs the project |

## How the pieces fit together

A request travels through three layers:

```text
Postman / Browser
      │  HTTP request (GET, POST, PUT, DELETE)
      ▼
StudentController   → receives HTTP requests and returns responses
      │
      ▼
StudentService      → holds the logic (find, create, update, delete)
      │
      ▼
StudentRepository   → talks to the database (Spring Data JPA)
      │
      ▼
PostgreSQL (student_db → student table)
```

The `Student` **entity** is the Java class that maps to the `student` table.

## Final API

By the end of Phase 1, your API will behave like this:

| Operation | Method | Endpoint | Success | Student missing |
|-----------|--------|----------|---------|-----------------|
| Create | <span class="method post">POST</span> | `/students` | `201 Created` | — |
| Read all | <span class="method get">GET</span> | `/students` | `200 OK` | — |
| Read one | <span class="method get">GET</span> | `/students/{id}` | `200 OK` | `404 Not Found` |
| Update | <span class="method put">PUT</span> | `/students/{id}` | `200 OK` | `404 Not Found` |
| Delete | <span class="method delete">DELETE</span> | `/students/{id}` | `204 No Content` | `404 Not Found` |

## Final project structure

```text
student-api
├── src
│   └── main
│       ├── java
│       │   └── com.example.student_api
│       │       ├── StudentApiApplication.java
│       │       ├── controller
│       │       │   └── StudentController.java
│       │       ├── entity
│       │       │   └── Student.java
│       │       ├── repository
│       │       │   └── StudentRepository.java
│       │       └── service
│       │           └── StudentService.java
│       └── resources
│           └── application.properties
├── pom.xml
├── mvnw
└── mvnw.cmd
```

## Phase 1 checklist

- [ ] [Environment setup](phase-1/01-environment-setup.md): project folder, Java check
- [ ] [Generate the project](phase-1/02-generate-project.md) with Spring Initializr
- [ ] [Open it in VS Code](phase-1/03-open-in-vscode.md)
- [ ] [Run it for the first time](phase-1/04-first-run.md)
- [ ] [Create the PostgreSQL database](phase-1/05-create-database.md)
- [ ] [Connect Spring Boot to PostgreSQL](phase-1/06-connect-postgresql.md)
- [ ] [Create the Student entity](phase-1/07-student-entity.md)
- [ ] [Create the repository](phase-1/08-repository.md)
- [ ] [Create the service layer](phase-1/09-service-layer.md)
- [ ] [Create the controller and first GET](phase-1/10-controller.md)
- [ ] [Create API (POST)](phase-1/11-create-api.md)
- [ ] [Read APIs (GET all, GET by ID)](phase-1/12-read-apis.md)
- [ ] [Update API (PUT)](phase-1/13-update-api.md)
- [ ] [Delete API (DELETE)](phase-1/14-delete-api.md)
- [ ] [Return 404 when a student doesn't exist](phase-1/15-not-found.md)
- [ ] [Return proper HTTP status codes](phase-1/16-status-codes.md)
- [ ] [404 for update and delete](phase-1/17-not-found-update-delete.md)

?> Want to see everything at once? The **[Final Code](phase-1/final-code.md)** page has every file in its finished state.

Ready? Start with **[1. Environment Setup](phase-1/01-environment-setup.md)** →
