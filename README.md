# Spring Boot Guide

Welcome! This is a friendly, hands-on guide to building backend APIs with **Spring Boot**. It is split into **phases**. Each phase builds on the one before it, and every phase is broken into small steps you can finish one at a time.

Every page follows the same pattern:

- **What we'll do**: the goal of the step in plain English.
- **The steps**: commands and code you can copy.
- **What this means**: a short explanation of the new idea.
- **✅ Checkpoint**: what you should see before you move on.

## Phases

| Phase | Topic | Status |
|-------|-------|--------|
| **[Phase 1](phase-1/README.md)** | Student Management CRUD API with Spring Web, Spring Data JPA and PostgreSQL | ✅ Available |
| **[Phase 2](phase-2/README.md)** | Validation, custom error messages, global exception handling, DTOs and duplicate-email protection | ✅ Available |
| **[Phase 3](phase-3/README.md)** | @Transactional, pagination, sorting, Spring Security basics, BCrypt signup and login | ✅ Available |
| Next phases | JWT access/refresh tokens, protected endpoints, USER/ADMIN roles and more | 🚧 Coming soon |

## The golden rule

?> **One step at a time.** Finish a step, check that it works, and only then move on. If you hit an error, stop and fix it before continuing. Most problems are easy to fix while they are new.

## What you need

- **Java 17 or later** (this guide uses Java 25)
- **PostgreSQL** with **pgAdmin** or `psql`
- **VS Code** with the *Extension Pack for Java*
- **Postman** (or any HTTP client) for testing the API

You **don't** need to install Maven. Every Spring Boot project comes with the Maven Wrapper (`mvnw`), which downloads the right Maven version for you.

## About this site

This site is built with [Docsify](https://docsify.js.org/). There is no build step: Docsify reads the Markdown files and shows them as web pages. It is hosted for free on **GitHub Pages**.

Ready? Start with **[Phase 1 — Overview](phase-1/README.md)** →
