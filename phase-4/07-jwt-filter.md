# 7. JWT Filter

> Read the token from every request and tell Spring Security who the user is.

## What is a filter?

Every request passes through a chain of **filters** before it reaches a controller. Spring Security is itself a set of filters. We'll add our own:

```text
HTTP request → JwtAuthenticationFilter → (other security filters) → authorization → controller
```

## Step 1 — Create the filter

Create `src/main/java/com/example/student_api/config/JwtAuthenticationFilter.java`:

```java
package com.example.student_api.config;

import com.example.student_api.service.JwtService;
import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

import java.io.IOException;

@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private final JwtService jwtService;

    public JwtAuthenticationFilter(JwtService jwtService) {
        this.jwtService = jwtService;
    }

    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain)
            throws ServletException, IOException {

        String authorizationHeader =
                request.getHeader("Authorization");

        if (authorizationHeader == null ||
                !authorizationHeader.startsWith("Bearer ")) {

            filterChain.doFilter(request, response);
            return;
        }

        String token =
                authorizationHeader.substring(7);

        if (jwtService.validateToken(token)) {

            String email =
                    jwtService.extractEmail(token);

            System.out.println("Authenticated user: " + email);
        }

        filterChain.doFilter(request, response);
    }
}
```

### What it does

| Code | Meaning |
|------|---------|
| `OncePerRequestFilter` | A Spring filter that runs exactly once per request |
| `request.getHeader("Authorization")` | Reads a header like `Bearer eyJhbGci...` |
| `startsWith("Bearer ")` | No Bearer token? Let the request continue; Spring Security decides later |
| `substring(7)` | Cuts off `"Bearer "` (7 characters), leaving just the token |
| `filterChain.doFilter(...)` | Passes the request on to the next filter |

!> **This filter doesn't authenticate anyone yet.** It only prints the email. We connect it to Spring Security in the next two steps.

Restart the app.

## Step 2 — Put the user into the SecurityContext

The filter knows the token belongs to `alice@example.com`, but Spring Security doesn't. We tell it through the **SecurityContext**: the place where Spring Security keeps "who is making this request".

In `JwtAuthenticationFilter.java`, add these imports:

```java
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
```

Replace:

```java
if (jwtService.validateToken(token)) {

    String email =
            jwtService.extractEmail(token);

    System.out.println("Authenticated user: " + email);
}
```

with:

```java
if (jwtService.validateToken(token)) {

    String email =
            jwtService.extractEmail(token);

    UsernamePasswordAuthenticationToken authentication =
            new UsernamePasswordAuthenticationToken(
                    email,
                    null,
                    java.util.Collections.emptyList()
            );

    SecurityContextHolder
            .getContext()
            .setAuthentication(authentication);

    System.out.println("Authenticated user: " + email);
}
```

| Argument | Meaning |
|----------|---------|
| `email` | Who the user is (the "principal") |
| `null` | Their password: we don't need it, the token already proved who they are |
| `Collections.emptyList()` | Their roles. None yet; we add roles on [page 12](phase-4/12-roles.md) |

The key line is `SecurityContextHolder.getContext().setAuthentication(authentication)`. It tells Spring Security: *"this request is authenticated."*

```text
JWT → validate → get email → create Authentication → SecurityContext → "this request is authenticated"
```

Restart the app.

## Step 3 — Add the filter to the security chain

The filter exists, but `SecurityConfig` doesn't use it yet. Replace `SecurityConfig.java` with:

```java
package com.example.student_api.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

@Configuration
public class SecurityConfig {

    @Autowired
    private JwtAuthenticationFilter jwtAuthenticationFilter;

    @Bean
    public SecurityFilterChain securityFilterChain(
            HttpSecurity http) throws Exception {

        http
                .csrf(csrf -> csrf.disable())
                .authorizeHttpRequests(auth -> auth
                        .requestMatchers("/students").authenticated()
                        .anyRequest().permitAll()
                )
                .httpBasic(customizer -> {})
                .addFilterBefore(
                        jwtAuthenticationFilter,
                        UsernamePasswordAuthenticationFilter.class
                );

        return http.build();
    }
}
```

`.addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class)` runs our filter **early**, before Spring Security's own username/password filter, so the user is already in the SecurityContext when the authorization rules are checked.

```text
HTTP request → JwtAuthenticationFilter → validate JWT → SecurityContext
             → Spring Security authorization → allow / reject
```

?> `@Autowired` on a field is another way to inject a bean. Constructor injection (what we use everywhere else) is generally preferred, but both work.

We still have Basic Auth turned on from Phase 3. We remove it on the next page.

Restart the app.

## Step 4 — Test with a Bearer token

1. Log in to get a **fresh** access token: <span class="method post">POST</span> `/auth/login`.
2. Copy the `accessToken`.
3. Send <span class="method get">GET</span> `http://localhost:8080/students`. In Bruno's **Auth** tab choose **Bearer Token** and paste the token.

You get **`200 OK`** with your students, and the Spring Boot console shows:

```text
Authenticated user: alice@example.com
```

?> Bruno's **Bearer Token** option sends the header `Authorization: Bearer <your token>` for you.

```text
JWT → JwtAuthenticationFilter → token validated → alice@example.com
    → SecurityContext → Spring Security → GET /students → 200 OK
```

We've moved from Basic Authentication to **JWT authentication**. 🎉

## ✅ Checkpoint

- [ ] `config/JwtAuthenticationFilter.java` exists and sets the SecurityContext
- [ ] `SecurityConfig` adds the filter with `addFilterBefore`
- [ ] `GET /students` with a Bearer token returns `200`, and the console shows the email

Next: **[8. Remove Basic Auth](phase-4/08-remove-basic-auth.md)** →
