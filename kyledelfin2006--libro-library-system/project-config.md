---
trigger: always_on
description: Provides Spring MVC, embedded HTTP server support, JSON serialization through Jackson, controllers, routing annotations, `ResponseEntity`, and web exception types. It implements the entire HTTP adapter around the book use cases.
---

# AGENTS.md

## Purpose

This file is the working guide for AI coding agents and human contributors to the Libro: Library API System. Read it before changing the project. It documents the repository as it exists, the intended architecture, the contracts between layers, and the checks expected before a contribution is considered complete.

The application is a single-module Spring Boot REST API for managing a library's book collection. It exposes CRUD, search, pagination, sorting, price filtering, aggregation, and health endpoints backed by PostgreSQL.

## Project Snapshot

| Concern | Current choice                                                          |
|---|-------------------------------------------------------------------------|
| Language | Java 25                                                                 |
| Framework | Spring Boot 4.1.0                                                       |
| Build | Maven, producing an executable Spring Boot JAR                          |
| HTTP layer | Spring MVC                                                              |
| Persistence | Spring Data JPA and Hibernate                                           |
| Database | PostgreSQL 18                                                           |
| Schema management | Flyway SQL migrations                                                   |
| Validation | Jakarta Bean Validation                                                 |
| Security | Spring Security filter chain; all requests currently permitted and CSRF disabled |
| Boilerplate reduction | Lombok 1.18.46                                                          |
| Testing | JUnit 5, Mockito, Jakarta Validator                                     |
| Coverage | JaCoCo report during Maven `verify`                                     |
| Containers | Dockerfile plus Docker Compose                                          |

The Maven coordinates are `kyle.com:library-api-system:1.0-SNAPSHOT`. There is no Maven Wrapper in the repository, so local commands require a compatible `mvn` installation unless Maven is added or supplied by the development environment.

## Repository Map

```text
library-api-system/
|-- AGENTS.md
|-- README.md
|-- pom.xml
|-- Dockerfile
|-- docker-compose.yml
|-- envFileExample
`-- src/
    |-- main/
    |   |-- java/app/
    |   |   |-- LibraryApplication.java
    |   |   |-- auth/
    |   |   |   `-- SecurityConfig.java
    |   |   |-- book/
    |   |   |   |-- controller/BookAPI.java
    |   |   |   |-- dto/
    |   |   |   |   |-- BookRequestDTO.java
    |   |   |   |   |-- BookResponseDTO.java
    |   |   |   |   `-- LibraryStatisticsDTO.java
    |   |   |   |-- entity/Book.java
    |   |   |   |-- exceptions/BookNotFoundException.java
    |   |   |   |-- mapper/BookMapper.java
    |   |   |   |-- repository/BookRepository.java
    |   |   |   `-- service/BookService.java
    |   |   `-- global/
    |   |       |-- exceptions/GlobalExceptionHandler.java
    |   |       `-- responses/
    |   |           |-- ApiResponse.java
    |   |           `-- ErrorResponse.java
    |   `-- resources/
    |       |-- application.properties
    |       `-- db/migration/
    |           |-- V1_create_books_table.sql
    |           |-- V2_create_users_table.sql
    |           `-- V3__add_created_at_to_books.sql
    `-- test/
        |-- java/unit/
        |   |-- BookMapperTest.java
        |   |-- BookServiceTest.java
        |   |-- BookTest.java
        |   `-- GlobalExceptionHandlerTest.java
        `-- resources/
            |-- junit-platform.properties
            `-- logback-test.xml
```

Keep production code below the root `app` package. `LibraryApplication` sits at that root so Spring's default component scan discovers controllers, services, repositories, mappers, advice, and configuration beneath it.

## Runtime Architecture

The standard request path is:

```text
HTTP client
  -> Spring Security filter chain
  -> BookAPI controller
  -> BookService
  -> BookRepository
  -> Hibernate/JPA
  -> PostgreSQL
```

The response path generally converts `Book` entities to DTOs through `BookMapper`. Exceptions escape their originating layer and are converted to JSON by `GlobalExceptionHandler`.

### Application entry point

`app.LibraryApplication` uses `@SpringBootApplication`, which combines configuration, auto-configuration, and component scanning. Do not move it below a feature package unless component scanning is configured explicitly.

### Controller layer

`BookAPI` owns HTTP concerns only:

- Base route: `/app/books`.
- Parses path variables, query parameters, pagination, and JSON bodies.
- Applies `@Valid` to complete create and replace payloads.
- Deliberately does not apply `@Valid` to PATCH payloads because omitted fields are represented by `null`.
- Delegates business rules to `BookService`.
- Uses `BookMapper` to prevent entities from becoming the public API representation.
- Chooses HTTP status codes and response envelopes.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kyledelfin2006/libro-library-system](https://github.com/kyledelfin2006/libro-library-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
