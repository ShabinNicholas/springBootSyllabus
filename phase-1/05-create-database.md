# 5. Create the Database

> Covers **Steps 5–6**: set up PostgreSQL and create the `student_db` database.

## What we'll do

Our API needs somewhere to store students. We'll create a PostgreSQL database called **`student_db`**.

## Step 6.1 — Open psql

In PowerShell:

```powershell
psql -h 127.0.0.1 -p 5432 -U postgres
```

It asks for your password:

```text
Password for user postgres:
```

Enter the password you created when you installed PostgreSQL.

?> The password characters don't show while you type. That's normal.

If it works, you'll see:

```text
psql (18.x)
Type "help" for help.

postgres=#
```

## Step 6.2 — Create the database

Inside psql:

```sql
CREATE DATABASE student_db;
```

You should get:

```text
CREATE DATABASE
```

## Step 6.3 — Verify it

List all databases:

```sql
\l
```

`student_db` should be in the list.

## Step 6.4 — Connect to it

```sql
\c student_db
```

```text
You are now connected to database "student_db" as user "postgres".
```

!> **Don't create any tables yourself.** Spring Boot and JPA/Hibernate will create the tables for us.

?> You can type `\q` to leave psql.

## ✅ Checkpoint

- [ ] `student_db` shows up in `\l`
- [ ] `\c student_db` connects successfully

Next: **[6. Connect to PostgreSQL](phase-1/06-connect-postgresql.md)** →
