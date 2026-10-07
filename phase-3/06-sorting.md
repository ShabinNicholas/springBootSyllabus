# 6. Sorting

> Let the client choose the order of results, with no code changes.

## Step 1 — Pageable already sorts

Good news: because we already take a `Pageable`, **sorting works already**. We don't need to change the repository, the service or the controller.

<span class="method get">GET</span> `/students?page=0&size=5&sort=name,asc`

```text
page  = 0
size  = 5
sort  = name
order = ascending
```

<span class="method get">GET</span> `/students?page=0&size=5&sort=age,desc`

```text
sort by age, highest → lowest
```

Spring puts the sort information into the same `Pageable` object, and passes it down the existing flow:

```text
Controller → Pageable → Service → studentRepository.findAll(pageable) → Database
```

So `Pageable` handles **both pagination and sorting**.

## Step 2 — Sort by name, A → Z

<span class="method get">GET</span> `http://localhost:8080/students?page=0&size=10&sort=name,asc`

The names come back in alphabetical order, for example:

```text
Alice Updated
David
Kevin
Kevin Updated
Michael Updated
No Email
Robert
Sarah Updated
Shabin
Test User
```

Spring read this as: page 0, size 10, sort field `name`, direction ASC. We didn't write any sorting code.

?> The exact order follows your database's sorting rules (for example, how it compares upper and lower case).

## Step 3 — Sort by name, Z → A

<span class="method get">GET</span> `http://localhost:8080/students?page=0&size=10&sort=name,desc`

```text
Test User
Shabin
Sarah Updated
Robert
No Email
Michael Updated
Kevin Updated
Kevin
David
Alice Updated
```

```text
sort=name,asc   → A → Z
sort=name,desc  → Z → A
```

## Step 4 — Sort by a number

<span class="method get">GET</span> `http://localhost:8080/students?page=0&size=10&sort=age,asc`

Students come back from lowest to highest age:

```text
22, 22, 22, 24, 24, 24, 25, 25, 26, 27
```

## Bonus — Sort by more than one field

Repeat the `sort` parameter:

<span class="method get">GET</span> `/students?page=0&size=10&sort=age,asc&sort=name,asc`

1. Sort by **age** ascending.
2. Students with the **same age** are then sorted by **name** ascending.

So the three 22-year-olds come back as:

```text
Alice Updated
David
Test User
```

This is called **secondary sorting**.

!> **Use real field names.** `sort` must match a field on the `Student` entity (`id`, `name`, `email`, `age`). Something like `sort=banana,asc` makes Spring throw an error, and you get `500 Internal Server Error` because our `GlobalExceptionHandler` doesn't handle that exception.

## What you've learned so far

```text
CRUD → HTTP status codes → Validation → Global exception handling
     → DTOs → Database constraints → @Transactional → Pagination → Sorting
```

?> **Search and filtering** (for example `GET /students?name=Kevin`) is another common feature. We'll come back to it and custom queries later. Next we start **authentication**.

## ✅ Checkpoint

- [ ] `sort=name,asc` and `sort=name,desc` reverse the order
- [ ] `sort=age,asc` sorts by age
- [ ] You tried two `sort` parameters together

Next: **[7. Spring Security](phase-3/07-spring-security.md)** →
