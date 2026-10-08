# Phase 4 — Final Code

Every file that changed or was added in Phase 4, in its **finished state**. Files not listed here are the same as in the [Phase 3 Final Code](phase-3/final-code.md) (for example `StudentController`, `StudentService`, `UserRepository`, `PasswordConfig` and the Phase 3 DTOs).

?> If your package name isn't `com.example.student_api`, change the `package` and `import` lines to match your project.

## pom.xml (added dependencies)

Inside `<dependencies>`:

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.13.0</version>
</dependency>

<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.13.0</version>
    <scope>runtime</scope>
</dependency>

<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.13.0</version>
    <scope>runtime</scope>
</dependency>
```

## application.properties (added line)

```properties
jwt.secret=my-super-secret-key-for-student-api-123456789
```

## Database changes

```sql
-- Users created before roles existed
UPDATE users SET role = 'USER' WHERE role IS NULL;

-- Your admin account (sign it up first)
UPDATE users SET role = 'ADMIN' WHERE email = 'admin@example.com';
```

## config/SecurityConfig.java

```java
package com.example.student_api.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
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
                .addFilterBefore(
                        jwtAuthenticationFilter,
                        UsernamePasswordAuthenticationFilter.class
                );

        return http.build();
    }
}
```

!> `GET /students/{id}` is still public here. See the warning on [page 13](phase-4/13-role-rules.md) for the one-line fix.

## config/JwtAuthenticationFilter.java

```java
package com.example.student_api.config;

import com.example.student_api.service.JwtService;
import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
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
                !authorizationHeader.startsWith("Bearer ") ||
                authorizationHeader.length() <= 7) {

            filterChain.doFilter(request, response);
            return;
        }

        String token =
                authorizationHeader.substring(7);

        if (jwtService.validateToken(token)) {

            String tokenType =
                    jwtService.extractTokenType(token);

            if ("access".equals(tokenType)) {

                String email = jwtService.extractEmail(token);

                String role = jwtService.extractRole(token);

                UsernamePasswordAuthenticationToken authentication =
                        new UsernamePasswordAuthenticationToken(
                                email,
                                null,
                                java.util.Collections.singletonList(
                                        new org.springframework.security.core.authority.SimpleGrantedAuthority(
                                                "ROLE_" + role
                                        )
                                )
                        );

                SecurityContextHolder
                        .getContext()
                        .setAuthentication(authentication);

                System.out.println("Authenticated user: " + email);
            }
        }

        filterChain.doFilter(request, response);
    }
}
```

## service/JwtService.java

```java
package com.example.student_api.service;

import com.example.student_api.entity.RefreshToken;
import com.example.student_api.exception.InvalidTokenException;
import com.example.student_api.repository.RefreshTokenRepository;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.security.Keys;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;

import javax.crypto.SecretKey;
import java.nio.charset.StandardCharsets;
import java.util.Date;

@Service
public class JwtService {

    @Value("${jwt.secret}")
    private String secretKey;

    private final RefreshTokenRepository refreshTokenRepository;

    public JwtService(RefreshTokenRepository refreshTokenRepository) {
        this.refreshTokenRepository = refreshTokenRepository;
    }

    public String generateAccessToken(String email, String role) {

        SecretKey key = Keys.hmacShaKeyFor(
                secretKey.getBytes(StandardCharsets.UTF_8)
        );

        return Jwts.builder()
                .subject(email)
                .claim("type", "access")
                .claim("role", role)
                .issuedAt(new Date())
                .expiration(new Date(
                        System.currentTimeMillis() + 15 * 60 * 1000
                ))
                .signWith(key)
                .compact();
    }

    public String generateRefreshToken(String email, String role) {

        SecretKey key = Keys.hmacShaKeyFor(
                secretKey.getBytes(StandardCharsets.UTF_8)
        );

        return Jwts.builder()
                .subject(email)
                .claim("type", "refresh")
                .claim("role", role)
                .issuedAt(new Date())
                .expiration(new Date(
                        System.currentTimeMillis() + 7L * 24 * 60 * 60 * 1000
                ))
                .signWith(key)
                .compact();
    }

    public boolean validateToken(String token) {

        SecretKey key = Keys.hmacShaKeyFor(
                secretKey.getBytes(StandardCharsets.UTF_8)
        );

        try {

            Jwts.parser()
                    .verifyWith(key)
                    .build()
                    .parseSignedClaims(token);

            return true;

        } catch (Exception e) {

            return false;
        }
    }

    public String extractEmail(String token) {

        SecretKey key = Keys.hmacShaKeyFor(
                secretKey.getBytes(StandardCharsets.UTF_8)
        );

        return Jwts.parser()
                .verifyWith(key)
                .build()
                .parseSignedClaims(token)
                .getPayload()
                .getSubject();
    }

    public String extractTokenType(String token) {

        SecretKey key = Keys.hmacShaKeyFor(
                secretKey.getBytes(StandardCharsets.UTF_8)
        );

        return Jwts.parser()
                .verifyWith(key)
                .build()
                .parseSignedClaims(token)
                .getPayload()
                .get("type", String.class);
    }

    public String extractRole(String token) {

        SecretKey key = Keys.hmacShaKeyFor(
                secretKey.getBytes(StandardCharsets.UTF_8)
        );

        return Jwts.parser()
                .verifyWith(key)
                .build()
                .parseSignedClaims(token)
                .getPayload()
                .get("role", String.class);
    }

    public String refreshAccessToken(String refreshToken) {

        RefreshToken refreshTokenEntity =
                refreshTokenRepository.findByToken(refreshToken)
                        .orElseThrow(() ->
                                new InvalidTokenException("Invalid refresh token"));

        if (refreshTokenEntity.isRevoked()) {
            throw new InvalidTokenException("Refresh token has been revoked");
        }

        if (!validateToken(refreshToken)) {
            throw new InvalidTokenException("Invalid or expired refresh token");
        }

        String tokenType = extractTokenType(refreshToken);

        if (!"refresh".equals(tokenType)) {
            throw new InvalidTokenException("Invalid refresh token");
        }

        String email = extractEmail(refreshToken);
        String role = extractRole(refreshToken);

        return generateAccessToken(email, role);
    }

    public void revokeRefreshToken(String refreshToken) {

        RefreshToken refreshTokenEntity =
                refreshTokenRepository.findByToken(refreshToken)
                        .orElseThrow(() ->
                                new InvalidTokenException("Invalid refresh token"));

        refreshTokenEntity.setRevoked(true);

        refreshTokenRepository.save(refreshTokenEntity);
    }
}
```

## service/AuthService.java

```java
package com.example.student_api.service;

import com.example.student_api.dto.LoginRequest;
import com.example.student_api.dto.LoginResponse;
import com.example.student_api.dto.SignupRequest;
import com.example.student_api.dto.UserResponse;
import com.example.student_api.entity.RefreshToken;
import com.example.student_api.entity.User;
import com.example.student_api.repository.RefreshTokenRepository;
import com.example.student_api.repository.UserRepository;

import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;

import java.util.Date;

@Service
public class AuthService {

    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    private final JwtService jwtService;
    private final RefreshTokenRepository refreshTokenRepository;

    public AuthService(
            UserRepository userRepository,
            PasswordEncoder passwordEncoder,
            JwtService jwtService,
            RefreshTokenRepository refreshTokenRepository) {

        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
        this.jwtService = jwtService;
        this.refreshTokenRepository = refreshTokenRepository;
    }

    public UserResponse signup(SignupRequest signupRequest) {

        User user = new User();

        user.setName(signupRequest.getName());
        user.setEmail(signupRequest.getEmail());

        String hashedPassword =
                passwordEncoder.encode(signupRequest.getPassword());

        user.setPassword(hashedPassword);
        user.setRole("USER");

        User savedUser = userRepository.save(user);

        UserResponse response = new UserResponse();

        response.setId(savedUser.getId());
        response.setName(savedUser.getName());
        response.setEmail(savedUser.getEmail());

        return response;
    }

    public LoginResponse login(LoginRequest loginRequest) {

        User user = userRepository.findByEmail(loginRequest.getEmail())
                .orElseThrow(() -> new RuntimeException("Invalid email or password"));

        boolean passwordMatches = passwordEncoder.matches(
                loginRequest.getPassword(),
                user.getPassword()
        );

        if (!passwordMatches) {
            throw new RuntimeException("Invalid email or password");
        }

        String accessToken =
                jwtService.generateAccessToken(
                        user.getEmail(),
                        user.getRole()
                );

        String refreshToken =
                jwtService.generateRefreshToken(
                        user.getEmail(),
                        user.getRole()
                );

        RefreshToken refreshTokenEntity = new RefreshToken();

        refreshTokenEntity.setToken(refreshToken);
        refreshTokenEntity.setRevoked(false);
        refreshTokenEntity.setExpiryDate(
                new Date(System.currentTimeMillis() + 7L * 24 * 60 * 60 * 1000)
        );

        refreshTokenEntity.setUser(user);

        refreshTokenRepository.save(refreshTokenEntity);

        LoginResponse response = new LoginResponse();

        response.setAccessToken(accessToken);
        response.setRefreshToken(refreshToken);

        return response;
    }
}
```

## controller/AuthController.java

```java
package com.example.student_api.controller;

import com.example.student_api.dto.LoginRequest;
import com.example.student_api.dto.LoginResponse;
import com.example.student_api.dto.LogoutRequest;
import com.example.student_api.dto.RefreshTokenRequest;
import com.example.student_api.dto.SignupRequest;
import com.example.student_api.dto.UserResponse;
import com.example.student_api.service.AuthService;
import com.example.student_api.service.JwtService;

import org.springframework.web.bind.annotation.*;

import java.util.HashMap;
import java.util.Map;

@RestController
@RequestMapping("/auth")
public class AuthController {

    private final AuthService authService;
    private final JwtService jwtService;

    public AuthController(
            AuthService authService,
            JwtService jwtService) {

        this.authService = authService;
        this.jwtService = jwtService;
    }

    @PostMapping("/signup")
    public UserResponse signup(@RequestBody SignupRequest signupRequest) {
        return authService.signup(signupRequest);
    }

    @PostMapping("/login")
    public LoginResponse login(@RequestBody LoginRequest loginRequest) {
        return authService.login(loginRequest);
    }

    @PostMapping("/refresh")
    public LoginResponse refreshToken(
            @RequestBody RefreshTokenRequest request) {

        String newAccessToken =
                jwtService.refreshAccessToken(request.getRefreshToken());

        LoginResponse response = new LoginResponse();

        response.setAccessToken(newAccessToken);
        response.setRefreshToken(request.getRefreshToken());

        return response;
    }

    @PostMapping("/logout")
    public Map<String, String> logout(@RequestBody LogoutRequest request) {

        jwtService.revokeRefreshToken(request.getRefreshToken());

        Map<String, String> response = new HashMap<>();
        response.put("message", "Logout successful");

        return response;
    }
}
```

## controller/GlobalExceptionHandler.java

```java
package com.example.student_api.controller;

import java.util.HashMap;
import java.util.Map;

import com.example.student_api.exception.InvalidTokenException;
import org.springframework.dao.DataIntegrityViolationException;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import org.springframework.web.server.ResponseStatusException;

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public Map<String, String> handleValidationErrors(
            MethodArgumentNotValidException ex) {

        Map<String, String> errors = new HashMap<>();

        ex.getBindingResult().getFieldErrors().forEach(error ->
                errors.put(error.getField(), error.getDefaultMessage())
        );

        return errors;
    }

    @ExceptionHandler(ResponseStatusException.class)
    public ResponseEntity<Map<String, String>> handleResponseStatusException(
            ResponseStatusException ex) {

        Map<String, String> error = new HashMap<>();

        error.put("message", ex.getReason());

        return ResponseEntity
                .status(ex.getStatusCode())
                .body(error);
    }

    @ExceptionHandler(DataIntegrityViolationException.class)
    @ResponseStatus(HttpStatus.CONFLICT)
    public Map<String, String> handleDataIntegrityViolation(
            DataIntegrityViolationException ex) {

        Map<String, String> error = new HashMap<>();

        error.put("message", "Email already exists");

        return error;
    }

    @ExceptionHandler(InvalidTokenException.class)
    @ResponseStatus(HttpStatus.UNAUTHORIZED)
    public Map<String, String> handleInvalidToken(
            InvalidTokenException ex) {

        Map<String, String> error = new HashMap<>();
        error.put("message", ex.getMessage());

        return error;
    }
}
```

## exception/InvalidTokenException.java

```java
package com.example.student_api.exception;

public class InvalidTokenException extends RuntimeException {

    public InvalidTokenException(String message) {
        super(message);
    }
}
```

## entity/User.java

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

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private String email;

    private String password;

    private String role;

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

    public String getRole() {
        return role;
    }

    public void setRole(String role) {
        this.role = role;
    }
}
```

## entity/RefreshToken.java

```java
package com.example.student_api.entity;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.JoinColumn;
import jakarta.persistence.ManyToOne;

import java.util.Date;

@Entity
public class RefreshToken {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(length = 1000)
    private String token;

    private boolean revoked;

    private Date expiryDate;

    @ManyToOne
    @JoinColumn(name = "user_id")
    private User user;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getToken() {
        return token;
    }

    public void setToken(String token) {
        this.token = token;
    }

    public boolean isRevoked() {
        return revoked;
    }

    public void setRevoked(boolean revoked) {
        this.revoked = revoked;
    }

    public Date getExpiryDate() {
        return expiryDate;
    }

    public void setExpiryDate(Date expiryDate) {
        this.expiryDate = expiryDate;
    }

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }
}
```

## repository/RefreshTokenRepository.java

```java
package com.example.student_api.repository;

import com.example.student_api.entity.RefreshToken;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.Optional;

public interface RefreshTokenRepository
        extends JpaRepository<RefreshToken, Long> {

    Optional<RefreshToken> findByToken(String token);
}
```

## dto/LoginResponse.java

```java
package com.example.student_api.dto;

public class LoginResponse {

    private String accessToken;
    private String refreshToken;

    public String getAccessToken() {
        return accessToken;
    }

    public void setAccessToken(String accessToken) {
        this.accessToken = accessToken;
    }

    public String getRefreshToken() {
        return refreshToken;
    }

    public void setRefreshToken(String refreshToken) {
        this.refreshToken = refreshToken;
    }
}
```

## dto/RefreshTokenRequest.java

```java
package com.example.student_api.dto;

public class RefreshTokenRequest {

    private String refreshToken;

    public String getRefreshToken() {
        return refreshToken;
    }

    public void setRefreshToken(String refreshToken) {
        this.refreshToken = refreshToken;
    }
}
```

## dto/LogoutRequest.java

```java
package com.example.student_api.dto;

public class LogoutRequest {

    private String refreshToken;

    public String getRefreshToken() {
        return refreshToken;
    }

    public void setRefreshToken(String refreshToken) {
        this.refreshToken = refreshToken;
    }
}
```

## API quick reference

| Operation | Method | Endpoint | Auth | Body | Success | Errors |
|-----------|--------|----------|------|------|---------|--------|
| Sign up | <span class="method post">POST</span> | `/auth/signup` | — | `SignupRequest` | `200` + `UserResponse` | — |
| Log in | <span class="method post">POST</span> | `/auth/login` | — | `LoginRequest` | `200` + both tokens | `500` (wrong details) |
| Refresh | <span class="method post">POST</span> | `/auth/refresh` | — | `{"refreshToken"}` | `200` + new access token | `401` |
| Log out | <span class="method post">POST</span> | `/auth/logout` | — | `{"refreshToken"}` | `200` + message | `401` |
| List students | <span class="method get">GET</span> | `/students` | USER / ADMIN | — | `200` | `403` |
| Create student | <span class="method post">POST</span> | `/students` | ADMIN | `StudentRequest` | `201` | `400`, `403`, `409` |
| Update student | <span class="method put">PUT</span> | `/students/{id}` | ADMIN | `StudentRequest` | `200` | `400`, `403`, `404`, `409` |
| Delete student | <span class="method delete">DELETE</span> | `/students/{id}` | ADMIN | — | `204` | `403`, `404` |

"Auth" means `Authorization: Bearer <access token>` with that role.

## What's next?

More phases are coming soon. 🚀
