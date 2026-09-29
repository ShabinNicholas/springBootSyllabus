# 1. Environment Setup

> Covers **Step 1**: create the project folder and check Java and Maven.

## What we'll do

Before we write any code, we need a folder to keep the project in, and we need to make sure Java is installed.

## Step 1.1 — Create the project folder

Open **PowerShell**. You can use any location you like. This guide uses your **Documents** folder.

```powershell
cd $HOME\Documents
```

Create a folder and move into it:

```powershell
mkdir springboot-crud
cd springboot-crud
```

Check where you are:

```powershell
pwd
```

You should see something like:

```text
Path
----
C:\Users\YourName\Documents\springboot-crud
```

## Step 1.2 — Check Java

```powershell
java -version
```

You need a modern Java version, **Java 17 or later**. This guide uses **Java 25 LTS**.

## Step 1.3 — Check Maven

```powershell
mvn -version
```

If Maven is installed, you'll see something like:

```text
Apache Maven ...
Java version: 25...
```

?> **Maven not installed? That's fine.** We don't need to install Maven globally. Spring Boot projects include the **Maven Wrapper** (`mvnw` / `mvnw.cmd`). The wrapper downloads and uses the right Maven version for the project.

!> **Don't create any Java files yet.** This step is only about getting the environment ready.

## ✅ Checkpoint

- [ ] The `springboot-crud` folder exists and you are inside it
- [ ] `java -version` shows Java 17 or later
- [ ] You checked `mvn -version` (it's okay if Maven isn't found)

Next: **[2. Generate the Project](phase-1/02-generate-project.md)** →
