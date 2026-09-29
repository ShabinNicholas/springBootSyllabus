# 2. Generate the Project

> Covers **Step 2**: generate the Spring Boot project with Spring Initializr.

## What we'll do

We'll use **[Spring Initializr](https://start.spring.io/)**, the standard way to create a Spring Boot project. It gives us a ready-to-run project with the dependencies we choose.

## Step 2.1 — Choose the options

Open [start.spring.io](https://start.spring.io/) and set:

| Setting | Select |
|---------|--------|
| Project | **Maven** |
| Language | **Java** |
| Spring Boot | **latest stable version** (not SNAPSHOT or M/RC) |
| Group | `com.example` |
| Artifact | `student-api` |
| Name | `student-api` |
| Packaging | **Jar** |
| Java | **25** (or the version you have installed) |

## Step 2.2 — Add dependencies

Click **Add Dependencies** and add these three:

1. **Spring Web**: build REST APIs, runs on an embedded Tomcat server
2. **Spring Data JPA**: work with the database using Java objects
3. **PostgreSQL Driver**: lets Java connect to PostgreSQL

That's all for now. Your dependency list should show:

```text
Spring Web
Spring Data JPA
PostgreSQL Driver
```

## Step 2.3 — Generate and extract

Click **GENERATE**. A ZIP file will download.

Extract it into the folder you created in Step 1:

```text
Documents
└── springboot-crud
    └── student-api
```

So your project location is:

```text
C:\Users\YourName\Documents\springboot-crud\student-api
```

!> **Don't open or change the code yet.** Just generate and extract the project.

## ✅ Checkpoint

- [ ] The project was generated with Spring Web, Spring Data JPA and PostgreSQL Driver
- [ ] It is extracted to `springboot-crud\student-api`

Next: **[3. Open in VS Code](phase-1/03-open-in-vscode.md)** →
