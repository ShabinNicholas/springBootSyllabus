# 4. First Run

> Covers **Step 4**: run the Spring Boot application for the first time.

## What we'll do

Before building the CRUD API, we'll make sure the application itself starts. Maven isn't installed globally, so we use the **Maven Wrapper** that came with the project.

## Step 4.1 — Open the VS Code terminal

In VS Code: **Terminal → New Terminal**

Make sure the terminal is in the project folder:

```powershell
pwd
```

```text
C:\Users\YourName\Documents\springboot-crud\student-api
```

## Step 4.2 — Run Spring Boot

```powershell
.\mvnw.cmd spring-boot:run
```

?> The first run can take a while because Maven downloads all the dependencies. Later runs are much faster.

Wait until you see lines like:

```text
Tomcat started on port 8080
Started StudentApiApplication
```

That means the Spring Boot server is running. 🎉

?> **Tip:** In PowerShell, `.\mvnw spring-boot:run` and `.\mvnw.cmd spring-boot:run` do the same thing. Later pages use the shorter form.

## Step 4.3 — Test it in the browser

Open [http://localhost:8080](http://localhost:8080).

You'll probably see a **Whitelabel Error Page**. **That's okay for now.** We haven't created any endpoints yet, so Spring Boot has nothing to return for `/`.

## If something goes wrong

If you see an error instead of `Started StudentApiApplication`, **don't try random fixes**. Copy the error section and read it carefully, or ask for help with the exact output.

?> **Heads-up:** Because we added **Spring Data JPA**, Spring Boot may complain that it can't configure a database (for example `Failed to configure a DataSource`). That's expected until we connect PostgreSQL in [Step 6](phase-1/06-connect-postgresql.md).

## ✅ Checkpoint

- [ ] `Started StudentApiApplication` appears in the terminal
- [ ] `http://localhost:8080` shows the Whitelabel Error Page

Keep the terminal running for now. When you need to stop the app later, press **Ctrl + C**.

Next: **[5. Create the Database](phase-1/05-create-database.md)** →
