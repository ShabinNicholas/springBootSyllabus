# Phase 2 — Validation & DTOs

In [Phase 1](phase-1/README.md) we built a working Student CRUD API. It works, but it **trusts whatever the client sends**: an empty name, an invalid email, a 12-year-old student or a duplicate email all go straight into the database.

In this phase we make the API **safe and clean**:

- **Validate** incoming data (name required, email valid, age 18+)
- Return **clear error messages** instead of a bare `400 Bad Request`
- Handle errors in one place with a **global exception handler**
- Separate the API from the database with **DTOs** (Data Transfer Objects)
- Return a clean **404** message for missing students
- **Prevent duplicate emails** and return `409 Conflict`

?> **Before you start:** Phase 2 builds on the finished Phase 1 code. If you're unsure your project is up to date, compare it with the [Phase 1 Final Code](phase-1/final-code.md).

## New concepts

| Concept | What it does |
|---------|--------------|
| **Bean Validation** (`@NotBlank`, `@Email`, `@Min`) | Rules that describe what valid data looks like |
| **`@Valid`** | Tells Spring to check those rules before running the controller method |
| **`@RestControllerAdvice`** | One class that handles exceptions for **all** controllers |
| **`@ExceptionHandler`** | A method that turns a specific exception into a response |
| **Request DTO** (`StudentRequest`) | The shape of data the API **accepts** |
| **Response DTO** (`StudentResponse`) | The shape of data the API **returns** |
| **Unique constraint** | The database refuses duplicate values |

## How a request flows after Phase 2

```text
Client (Postman)
      │  JSON
      ▼
StudentRequest DTO   ← @Valid checks the rules here
      │                 invalid? → GlobalExceptionHandler → 400 + messages
      ▼
StudentController
      │
      ▼
StudentService       ← StudentRequest → Student entity
      │                 not found? → 404 + message
      ▼
StudentRepository → PostgreSQL
      │                 duplicate email? → 409 + message
      ▼
Student entity → StudentResponse DTO → JSON back to the client
```

## Final API behaviour

| Request | Result |
|---------|--------|
| Valid <span class="method post">POST</span> `/students` | `201 Created` |
| <span class="method post">POST</span> / <span class="method put">PUT</span> with invalid data | `400 Bad Request` + field messages |
| <span class="method get">GET</span> / <span class="method put">PUT</span> / <span class="method delete">DELETE</span> a missing student | `404 Not Found` + `{"message": "Student not found"}` |
| <span class="method post">POST</span> with an email that already exists | `409 Conflict` + `{"message": "Email already exists"}` |

## Final project structure

```text
com.example.student_api
├── StudentApiApplication.java
├── controller
│   ├── StudentController.java
│   └── GlobalExceptionHandler.java   ← new
├── dto                               ← new
│   ├── StudentRequest.java
│   └── StudentResponse.java
├── entity
│   └── Student.java
├── repository
│   └── StudentRepository.java
└── service
    └── StudentService.java
```

## Phase 2 checklist

- [ ] [Add the validation dependency](phase-2/01-validation-dependency.md)
- [ ] [First validation rule: name required](phase-2/02-first-validation.md)
- [ ] [Email and age validation](phase-2/03-email-age-validation.md)
- [ ] [Validate PUT, and @Email vs @NotBlank](phase-2/04-validate-put.md)
- [ ] [Custom validation messages](phase-2/05-custom-messages.md)
- [ ] [Global exception handler](phase-2/06-exception-handler.md)
- [ ] [Return 400 explicitly and test every rule](phase-2/07-validation-status.md)
- [ ] [Request DTO for POST](phase-2/08-request-dto.md)
- [ ] [Clean up the entity and use the DTO for PUT](phase-2/09-request-dto-put.md)
- [ ] [Response DTO for POST](phase-2/10-response-dto.md)
- [ ] [Response DTO for PUT and GET](phase-2/11-response-dto-all.md)
- [ ] [Clean 404 error messages](phase-2/12-not-found-handler.md)
- [ ] [Prevent duplicate emails (409)](phase-2/13-duplicate-emails.md)

?> Want to see everything at once? The **[Final Code](phase-2/final-code.md)** page has every file in its finished Phase 2 state.

Ready? Start with **[1. Validation Dependency](phase-2/01-validation-dependency.md)** →
