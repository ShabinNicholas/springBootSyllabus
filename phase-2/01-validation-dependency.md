# 1. Validation Dependency

> Covers **Step 26.1**: add Spring Boot's validation starter.

## What we'll do

Right now our API accepts anything. For example, this would create a student with **no name**:

```json
{
  "name": "",
  "email": "test@gmail.com",
  "age": 22
}
```

To stop that, we need **Bean Validation**: annotations like `@NotBlank`, `@Email` and `@Min` that describe what valid data looks like. They aren't included in the dependencies we picked in Phase 1, so we'll add them.

## Step 26.1 — Add the dependency

Open `pom.xml`. Find the `<dependencies>` section and add this inside it, next to the other `<dependency>` blocks:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

?> There's no `<version>` tag. That's normal: Spring Boot manages the version for you, just like the other starters.

Save `pom.xml`.

## Step 26.1.1 — Let Maven download it

VS Code usually notices the change and asks whether to **synchronize** or **reload** the Java project. Click **Yes / Always**.

?> If nothing happens, stop the app (**Ctrl + C**) and run `.\mvnw spring-boot:run` again. Maven downloads new dependencies when it starts.

## Step 26.1.2 — Check that the import works

Open any Java file (for example `Student.java`) and type this import near the top:

```java
import jakarta.validation.constraints.NotBlank;
```

If there's **no red underline**, the dependency is installed. You can leave the import there, because we'll use it on the next page.

!> **Don't change anything else in `Student.java` yet.**

## ✅ Checkpoint

- [ ] `spring-boot-starter-validation` is in `pom.xml`
- [ ] `import jakarta.validation.constraints.NotBlank;` shows no error

Next: **[2. First Validation Rule](phase-2/02-first-validation.md)** →
