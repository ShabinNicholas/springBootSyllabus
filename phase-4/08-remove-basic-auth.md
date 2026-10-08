# 8. Remove Basic Auth

> Turn off Basic Auth, make the filter handle bad headers safely, and see what happens with no token.

## Step 1 — Remove httpBasic

We now authenticate with JWTs, so we don't need Basic Auth. In `SecurityConfig.java`, delete this line:

```java
.httpBasic(customizer -> {})
```

The chain is now:

```java
http
        .csrf(csrf -> csrf.disable())
        .authorizeHttpRequests(auth -> auth
                .requestMatchers("/students").authenticated()
                .anyRequest().permitAll()
        )
        .addFilterBefore(
                jwtAuthenticationFilter,
                UsernamePasswordAuthenticationFilter.class
        );
```

So the API is moving towards:

```text
Signup   → public
Login    → public
Refresh  → public
Students → JWT required
```

Restart the app.

## Step 2 — Make the filter safe with bad headers

When testing without a token, the walkthrough this guide is based on got a `500 Internal Server Error`, caused by an incomplete `Authorization` header reaching `substring(7)`. Let's make the filter skip any header that's too short to contain a token.

In `JwtAuthenticationFilter.java`, change the first `if`:

```java
if (authorizationHeader == null ||
        !authorizationHeader.startsWith("Bearer ")) {
```

to:

```java
if (authorizationHeader == null ||
        !authorizationHeader.startsWith("Bearer ") ||
        authorizationHeader.length() <= 7) {
```

`"Bearer "` is exactly 7 characters. If the header is that long or shorter, there's no token after it, so we just let the request continue and Spring Security decides whether it's allowed.

?> **Check Bruno's Auth tab.** If the Auth type is still *Bearer Token* with an empty token field, Bruno may still send an `Authorization` header. To test "no token", set Auth to **No Auth**.

Restart the app.

## Step 3 — Test without a token

Send <span class="method get">GET</span> `http://localhost:8080/students` with **No Auth**.

You get **`403 Forbidden`**.

### Why 403 and not 401?

You might expect `401 Unauthorized`, because we're not logged in. But Spring Security only sends a `401` when an authentication method tells it how to ask for credentials (Basic Auth sends `401` with a `WWW-Authenticate` header, for example). We just removed Basic Auth, and our JWT filter doesn't tell Spring Security what to answer. So it falls back to its default: **`403`**.

```text
No JWT → nobody in the SecurityContext → /students needs authentication → 403 Forbidden
```

?> **Want a proper 401?** Add this to the `http` chain in `SecurityConfig`:
> ```java
> .exceptionHandling(e -> e.authenticationEntryPoint(
>         new HttpStatusEntryPoint(HttpStatus.UNAUTHORIZED)))
> ```
> with `import org.springframework.http.HttpStatus;` and `import org.springframework.security.web.authentication.HttpStatusEntryPoint;`. This guide keeps the default `403` to match the rest of the steps.

## Step 4 — Test with a token

Send the same request with **Bearer Token** and a fresh access token: **`200 OK`**.

```text
GET /students
Without JWT → ❌ 403
With JWT    → ✅ 200
```

?> Access tokens expire after **15 minutes**. If a request that used to work suddenly returns `403`, log in again for a fresh token.

## ✅ Checkpoint

- [ ] `.httpBasic(...)` is gone from `SecurityConfig`
- [ ] The filter checks `authorizationHeader.length() <= 7`
- [ ] No token → `403`, valid token → `200`

Next: **[9. Refresh Endpoint](phase-4/09-refresh-endpoint.md)** →
