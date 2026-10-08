# Phase 4 — JWT, Roles & Logout

In [Phase 3](phase-3/README.md) we stored users in the database with BCrypt passwords, and `/auth/login` returned `true` or `false`. In this phase login returns **real tokens**, and the API uses them to decide who you are and what you're allowed to do.

- **Access tokens** (15 minutes) for calling the API
- **Refresh tokens** (7 days) for getting a new access token without logging in again
- A **JWT filter** that checks the token on every request
- **Token types**, so a refresh token can't be used as an access token (and the other way round)
- **Roles** (`USER` and `ADMIN`) and rules about which role can do what
- **Logout**, with refresh tokens stored in PostgreSQL so they can be revoked

?> **Before you start:** Phase 4 builds on the finished Phase 3 code. If you're unsure your project is up to date, compare it with the [Phase 3 Final Code](phase-3/final-code.md).

?> **Step numbers:** like Phase 3, each page numbers its own steps (Step 1, Step 2, ...).

## How a request works after Phase 4

```text
POST /auth/login  (email + password)
        ↓
Check password with BCrypt
        ↓
Access Token (15 min)  +  Refresh Token (7 days, saved in the database)

GET /students
Authorization: Bearer <access token>
        ↓
JwtAuthenticationFilter
        ↓  valid?  type = access?
Read email + role from the token
        ↓
Spring Security: ROLE_USER / ROLE_ADMIN
        ↓
Authorization rules → allow (200) or deny (403)

POST /auth/refresh  (refresh token)  → new access token
POST /auth/logout   (refresh token)  → refresh token revoked in the database
```

## Final API behaviour

| Request | USER | ADMIN | No token |
|---------|------|-------|----------|
| <span class="method get">GET</span> `/students` | ✅ `200` | ✅ `200` | ❌ `403` |
| <span class="method post">POST</span> `/students` | ❌ `403` | ✅ `201` | ❌ `403` |
| <span class="method put">PUT</span> `/students/{id}` | ❌ `403` | ✅ `200` | ❌ `403` |
| <span class="method delete">DELETE</span> `/students/{id}` | ❌ `403` | ✅ `204` | ❌ `403` |
| <span class="method post">POST</span> `/auth/signup`, `/auth/login`, `/auth/refresh`, `/auth/logout` | public | public | public |

## Final project structure

```text
com.example.student_api
├── StudentApiApplication.java
├── config
│   ├── JwtAuthenticationFilter.java   ← new
│   ├── PasswordConfig.java
│   └── SecurityConfig.java            (changed)
├── controller
│   ├── AuthController.java            (changed)
│   ├── GlobalExceptionHandler.java    (changed)
│   └── StudentController.java
├── dto
│   ├── LoginRequest.java
│   ├── LoginResponse.java             ← new
│   ├── LogoutRequest.java             ← new
│   ├── RefreshTokenRequest.java       ← new
│   ├── SignupRequest.java
│   ├── StudentPageResponse.java
│   ├── StudentRequest.java
│   ├── StudentResponse.java
│   └── UserResponse.java
├── entity
│   ├── RefreshToken.java              ← new
│   ├── Student.java
│   └── User.java                      (changed: role)
├── exception                          ← new
│   └── InvalidTokenException.java
├── repository
│   ├── RefreshTokenRepository.java    ← new
│   ├── StudentRepository.java
│   └── UserRepository.java
└── service
    ├── AuthService.java               (changed)
    ├── JwtService.java                ← new
    └── StudentService.java
```

## Phase 4 checklist

**Tokens**
- [ ] [JWT dependency and JwtService](phase-4/01-jwt-setup.md)
- [ ] [Generate an access token at login](phase-4/02-access-token.md)
- [ ] [LoginResponse DTO](phase-4/03-login-response.md)
- [ ] [Refresh token](phase-4/04-refresh-token.md)
- [ ] [Move the secret to application.properties](phase-4/05-secret-config.md)
- [ ] [Validate a token and read the email](phase-4/06-validate-token.md)

**Using the token**
- [ ] [JWT authentication filter](phase-4/07-jwt-filter.md)
- [ ] [Remove Basic Auth](phase-4/08-remove-basic-auth.md)
- [ ] [The refresh endpoint](phase-4/09-refresh-endpoint.md)
- [ ] [Access vs refresh token types](phase-4/10-token-types.md)
- [ ] [InvalidTokenException and 401](phase-4/11-invalid-token.md)

**Authorization**
- [ ] [Roles in the user and the token](phase-4/12-roles.md)
- [ ] [Role-based rules and an ADMIN user](phase-4/13-role-rules.md)

**Logout**
- [ ] [Logout (in memory)](phase-4/14-logout.md)
- [ ] [Store refresh tokens in PostgreSQL](phase-4/15-refresh-token-db.md)
- [ ] [Link refresh tokens to users](phase-4/16-token-user-link.md)

?> Want to see everything at once? The **[Final Code](phase-4/final-code.md)** page has every changed file in its finished Phase 4 state.

Ready? Start with **[1. JWT Setup](phase-4/01-jwt-setup.md)** →
