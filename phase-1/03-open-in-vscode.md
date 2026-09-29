# 3. Open in VS Code

> Covers **Step 3**: open the project and get to know the generated structure.

## What we'll do

Open the `student-api` project in VS Code and make sure it is recognised as a Spring Boot project.

## Step 3.1 — Go to the project in PowerShell

```powershell
cd $HOME\Documents\springboot-crud\student-api
dir
```

You should see files and folders like:

```text
.mvn
src
.gitignore
mvnw
mvnw.cmd
pom.xml
```

The important ones for now:

| File / folder | Purpose |
|---------------|---------|
| `src` | Where our Java code lives |
| `pom.xml` | Project dependencies and configuration |
| `mvnw.cmd` | Maven Wrapper for Windows |
| `.mvn` | Maven Wrapper configuration |

## Step 3.2 — Open it in VS Code

From the same PowerShell window:

```powershell
code .
```

VS Code opens the `student-api` folder.

## Step 3.3 — Find the main class

In VS Code, expand:

```text
src
└── main
    └── java
        └── com.example.student_api
```

Open `StudentApiApplication.java`. It should look roughly like this:

```java
package com.example.student_api;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class StudentApiApplication {

    public static void main(String[] args) {
        SpringApplication.run(StudentApiApplication.class, args);
    }
}
```

?> **Check your package name.** Spring Initializr turns the artifact `student-api` into a package name. It's usually `com.example.student_api`, but it can be `com.example.studentapi`. Look at the first line of `StudentApiApplication.java`. This guide uses `com.example.student_api`. If yours is different, **use your own package name** in every file you create.

!> **Don't change anything yet.**

## Step 3.4 — Java support in VS Code

If VS Code asks you to install Java extensions, install **Extension Pack for Java** by Microsoft. If you already have Java extensions installed, that's fine.

## ✅ Checkpoint

- [ ] The project opens in VS Code
- [ ] `pom.xml` is visible
- [ ] `StudentApiApplication.java` is visible
- [ ] No major Java or project errors are showing

Next: **[4. First Run](phase-1/04-first-run.md)** →
