# 13. Role Rules

> Decide which role can call which endpoint, and create an ADMIN user to test with.

The goal:

| Request | USER | ADMIN |
|---------|------|-------|
| <span class="method get">GET</span> `/students` | ✅ | ✅ |
| <span class="method post">POST</span> `/students` | ❌ | ✅ |
| <span class="method put">PUT</span> `/students/{id}` | ❌ | ✅ |
| <span class="method delete">DELETE</span> `/students/{id}` | ❌ | ✅ |

## Step 1 — USER and ADMIN can view students

In `SecurityConfig.java`, change:

```java
.requestMatchers("/students").authenticated()
```

to:

```java
.requestMatchers("/students").hasAnyRole("USER", "ADMIN")
```

`hasAnyRole("USER", "ADMIN")` means: *allow the request if the user has `ROLE_USER` **or** `ROLE_ADMIN`.* Spring adds the `ROLE_` prefix for you.

Restart and send <span class="method get">GET</span> `/students` with Alice's access token: **`200 OK`**.

## Step 2 — Separate rules per HTTP method

`.requestMatchers("/students")` applies to **every** method (GET, POST...). We need different rules for GET and POST, so we pass the method too.

In `SecurityConfig.java`, add this import:

```java
import org.springframework.http.HttpMethod;
```

and change the rules to:

```java
.authorizeHttpRequests(auth -> auth
        .requestMatchers(
                HttpMethod.GET,
                "/students"
        ).hasAnyRole("USER", "ADMIN")

        .requestMatchers(
                HttpMethod.POST,
                "/students"
        ).hasRole("ADMIN")

        .anyRequest().permitAll()
)
```

`HttpMethod.GET` means *"apply this rule only to GET requests"*.

```text
GET  /students → USER or ADMIN → allowed ✅
POST /students → ADMIN only    → USER denied ❌, ADMIN allowed ✅
```

Restart the app.

## Step 3 — Test USER creating a student

With **Alice's** access token, send <span class="method post">POST</span> `/students`:

```json
{
  "name": "Test Student",
  "email": "teststudent@example.com",
  "age": 22
}
```

**`403 Forbidden`**:

```text
Alice → authenticated ✅ → ROLE_USER → POST /students needs ROLE_ADMIN → 403 ❌
```

| Status | Meaning |
|--------|---------|
| `401 Unauthorized` | Authentication problem: *who are you?* |
| `403 Forbidden` | Authorization problem: *I know who you are, but you're not allowed* |

## Step 4 — Create an ADMIN user safely

We won't let signup accept a role (anyone could become admin). For this learning project, we promote a user **directly in the database**.

**1. Sign up a normal user:** <span class="method post">POST</span> `/auth/signup`:

```json
{
  "name": "Admin User",
  "email": "admin@example.com",
  "password": "admin123"
}
```

It starts as `role = USER`, like everyone.

**2. Promote it in PostgreSQL:**

```sql
UPDATE users
SET role = 'ADMIN'
WHERE email = 'admin@example.com';
```

```sql
SELECT id, name, email, role
FROM users
WHERE email = 'admin@example.com';
```

You should see `ADMIN`.

## Step 5 — Log in as ADMIN

<span class="method post">POST</span> `/auth/login`:

```json
{
  "email": "admin@example.com",
  "password": "admin123"
}
```

Decode the new access token. It should contain `"role": "ADMIN"`.

?> If you logged in as this user **before** the `UPDATE`, that old token still says `USER`. Always log in again after changing a role.

## Step 6 — Test ADMIN creating a student

With the **ADMIN** access token, send <span class="method post">POST</span> `/students`:

```json
{
  "name": "Admin Created Student",
  "email": "adminstudent@example.com",
  "age": 22
}
```

**`201 Created`** with the new student. Note its `id`; we delete it in Step 9.

```text
ADMIN token → role = ADMIN → ROLE_ADMIN → POST /students → hasRole("ADMIN") → allowed ✅
```

## Step 7 — Only ADMIN can update

Add a PUT rule between the POST rule and `.anyRequest()`:

```java
.requestMatchers(
        HttpMethod.PUT,
        "/students/{id}"
).hasRole("ADMIN")
```

`"/students/{id}"` matches any ID: `/students/12`, `/students/20` and so on.

Restart, then with **Alice's** token send <span class="method put">PUT</span> `/students/12`:

```json
{
  "name": "User Trying Update",
  "email": "userupdate@example.com",
  "age": 23
}
```

**`403 Forbidden`**. ✅

## Step 8 — Only ADMIN can delete

Add a DELETE rule after the PUT rule:

```java
.requestMatchers(
        HttpMethod.DELETE,
        "/students/{id}"
).hasRole("ADMIN")
```

The rules are now:

```java
.authorizeHttpRequests(auth -> auth
        .requestMatchers(
                HttpMethod.GET,
                "/students"
        ).hasAnyRole("USER", "ADMIN")

        .requestMatchers(
                HttpMethod.POST,
                "/students"
        ).hasRole("ADMIN")

        .requestMatchers(
                HttpMethod.PUT,
                "/students/{id}"
        ).hasRole("ADMIN")

        .requestMatchers(
                HttpMethod.DELETE,
                "/students/{id}"
        ).hasRole("ADMIN")

        .anyRequest().permitAll()
)
```

Restart the app.

## Step 9 — Test DELETE

**USER (optional):** <span class="method delete">DELETE</span> `/students/12` with Alice's token → **`403`**, and the student is still there.

**ADMIN:** <span class="method delete">DELETE</span> `/students/{id}` with the ADMIN token, using the ID of *Admin Created Student* from Step 6 → **`204 No Content`**.

Check with <span class="method get">GET</span> `/students/{id}`: `404` with `{"message": "Student not found"}`.

## Result

| Request | USER | ADMIN |
|---------|------|-------|
| <span class="method get">GET</span> `/students` | ✅ `200` | ✅ `200` |
| <span class="method post">POST</span> `/students` | ❌ `403` | ✅ `201` |
| <span class="method put">PUT</span> `/students/{id}` | ❌ `403` | ✅ `200` |
| <span class="method delete">DELETE</span> `/students/{id}` | ❌ `403` | ✅ `204` |

!> **One gap: `GET /students/{id}` is still public.** The GET rule matches only `/students`, and `/students/5` falls through to `.anyRequest().permitAll()`. To protect it, add this rule next to the other GET rule:
> ```java
> .requestMatchers(HttpMethod.GET, "/students/{id}").hasAnyRole("USER", "ADMIN")
> ```

?> **Rule order matters.** Spring checks rules from top to bottom and uses the **first** one that matches. That's why `.anyRequest()` must always come last.

?> **Another way: `@PreAuthorize`.** Instead of listing every rule in `SecurityConfig`, you can put `@PreAuthorize("hasRole('ADMIN')")` directly above a controller method. That needs method security turned on, and we'll look at it in a later phase.

## ✅ Checkpoint

- [ ] `SecurityConfig` has GET, POST, PUT and DELETE rules
- [ ] An ADMIN user exists and its token has `"role": "ADMIN"`
- [ ] USER gets `403` for POST/PUT/DELETE; ADMIN succeeds

Next: **[14. Logout](phase-4/14-logout.md)** →
