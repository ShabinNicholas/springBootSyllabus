# Phase 3 — Transactions, Paging & Security Basics

In [Phase 2](phase-2/README.md) we made the API safe and clean with validation, DTOs and a global exception handler. In this phase we learn three more things every real Spring Boot API needs, and take the first steps into security.

- **`@Transactional`**: treat several database operations as one unit of work
- **Pagination**: return students one page at a time, with page information
- **Sorting**: let the client choose the order (`sort=name,asc`)
- **Spring Security**: what it does as soon as you add it, and how `SecurityFilterChain` works
- **Users in the database**: a `User` entity, `UserRepository` and the `users` table
- **Signup**: hash passwords with **BCrypt** and never return them
- **Login**: check an email and password against the stored hash

?> **Before you start:** Phase 3 builds on the finished Phase 2 code. If you're unsure your project is up to date, compare it with the [Phase 2 Final Code](phase-2/final-code.md).

?> **Step numbers:** the original walkthrough restarted its step numbers several times in this phase. To keep things clear, each page here numbers its own steps (Step 1, Step 2, ...).

## The authentication roadmap

Security is a big topic, so we build it in stages. This phase covers the first five:

```text
1. Authentication vs Authorization   ✅ Phase 3
2. Add Spring Security               ✅ Phase 3
3. Password hashing (BCrypt)         ✅ Phase 3
4. Signup                            ✅ Phase 3
5. Login (verify password)           ✅ Phase 3
6. JWT access + refresh tokens       🚧 next phase
7. JWT validation                    🚧 next phase
8. Protect student endpoints         🚧 next phase
9. Roles: USER vs ADMIN              🚧 next phase
```

We deliberately **don't** jump straight to JWT. First you see what Spring Security does by default, so you understand *why* JWT is needed later instead of just copying code.

## API after Phase 3

| Request | Auth needed? | Result |
|---------|--------------|--------|
| <span class="method get">GET</span> `/students?page=0&size=5&sort=name,asc` | ✅ Basic Auth | `200` + a page of students |
| <span class="method post">POST</span> `/students` | ✅ Basic Auth | same as Phase 2 |
| <span class="method get">GET</span> / <span class="method put">PUT</span> / <span class="method delete">DELETE</span> `/students/{id}` | ❌ not yet (see [page 8](phase-3/08-security-config.md)) | same as Phase 2 |
| <span class="method post">POST</span> `/auth/signup` | ❌ | `200` + `{id, name, email}`, no password |
| <span class="method post">POST</span> `/auth/login` | ❌ | `true` or `false` (temporary, for learning) |

## Final project structure

```text
com.example.student_api
├── StudentApiApplication.java
├── config                         ← new
│   ├── PasswordConfig.java
│   └── SecurityConfig.java
├── controller
│   ├── AuthController.java        ← new
│   ├── GlobalExceptionHandler.java
│   └── StudentController.java
├── dto
│   ├── LoginRequest.java          ← new
│   ├── SignupRequest.java         ← new
│   ├── StudentPageResponse.java   ← new
│   ├── StudentRequest.java
│   ├── StudentResponse.java
│   └── UserResponse.java          ← new
├── entity
│   ├── Student.java
│   └── User.java                  ← new
├── repository
│   ├── StudentRepository.java
│   └── UserRepository.java        ← new
└── service
    ├── AuthService.java           ← new
    └── StudentService.java
```

## Phase 3 checklist

**Transactions**
- [ ] [@Transactional basics](phase-3/01-transactional.md)
- [ ] [See a rollback happen](phase-3/02-rollback-demo.md)
- [ ] [Where @Transactional goes, and readOnly](phase-3/03-readonly.md)

**Pagination & sorting**
- [ ] [Pagination with Pageable](phase-3/04-pagination.md)
- [ ] [A page response DTO](phase-3/05-page-response.md)
- [ ] [Sorting](phase-3/06-sorting.md)

**Security basics**
- [ ] [Add Spring Security](phase-3/07-spring-security.md)
- [ ] [SecurityConfig and Basic Auth](phase-3/08-security-config.md)
- [ ] [User entity and repository](phase-3/09-user-entity.md)
- [ ] [Signup with BCrypt](phase-3/10-signup.md)
- [ ] [Signup endpoint and CSRF](phase-3/11-signup-endpoint.md)
- [ ] [Don't return the password](phase-3/12-user-response.md)
- [ ] [Login](phase-3/13-login.md)

?> Want to see everything at once? The **[Final Code](phase-3/final-code.md)** page has every changed file in its finished Phase 3 state.

Ready? Start with **[1. @Transactional](phase-3/01-transactional.md)** →
