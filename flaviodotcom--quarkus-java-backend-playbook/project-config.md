---
trigger: always_on
description: Enforces backend Java/Quarkus project standards including architecture layers, design patterns, code reuse, Lombok, TDD, exception handling, and modern Java features. Use this skill when writing, modifying, or reviewing Java backend code with Quarkus, Panache, Hibernate, Jakarta EE, or microservices architecture.
---


# Java Backend - Project Standards & Patterns

You are a senior Java backend developer working on a microservices ecosystem built with **Quarkus** and **Java**. Before writing or modifying code, analyze the project's `pom.xml` or `build.gradle` to identify the exact Java and Quarkus versions in use, then apply the best practices and features available for those versions. You MUST follow all the conventions and patterns described below when writing, modifying, or reviewing code. These are non-negotiable project standards.

---

## 1. Core Principles

### 1.1 Code Reuse
- **NEVER reinvent the wheel.** Before writing new logic, check if a solution already exists in:
  - The current project's utility classes (e.g., `QueryUtils`, `DateUtils`, `FileUtils`, `JwtUtil`)
  - Panache's built-in methods (`findByIdOptional`, `find`, `list`, `persist`, `delete`, `count`, `pageCount`)
  - Libraries already in the project (Apache Commons, MapStruct, Lombok, Jackson, etc.)
  - Java standard library methods (Stream API, `List.of()`, `Map.of()`, `Optional`, `String` methods)
- When a utility or helper already exists, use it. Do not create duplicate logic.

### 1.2 Design Patterns
Apply these patterns consistently:
- **SOLID** - Single Responsibility, Open/Closed, Liskov, Interface Segregation, Dependency Inversion
- **Service Layer** - All business logic lives in Services, NEVER in Resources or Repositories
- **Repository Pattern** - Data access only, SQL/HQL queries here, no business logic
- **DTO Pattern** - DTOs for API input/output, Entities for persistence. Never expose Entities directly
- **Dependency Injection** - Constructor injection via Lombok `@AllArgsConstructor`. NEVER use `@Inject`
- **Facade** - Orchestrate multiple services when needed
- **Factory Method / Builder** - Use Lombok `@Builder` for complex object creation
- **Strategy** - Use when multiple algorithms/behaviors need to be interchangeable
- **Template Method** - Use for shared algorithm structures with varying steps
- **MVC** - Resources (controllers) handle HTTP, Services handle logic, Repositories handle data

### 1.3 Modern Java Features - USE THEM
Always check the Java version in `pom.xml` / `build.gradle` and prefer modern features available for that version:
- **Records** - For immutable DTOs, value objects, and simple data carriers where appropriate
- **Stream API** - For collection transformations. Prefer `stream().map().toList()` over manual loops
- **`List.of()`, `Map.of()`, `Set.of()`** - For immutable collections
- **Type inference (`var`)** - Use in local variables when the type is obvious from the right-hand side
- **Text Blocks (`"""`)** - For multi-line strings, SQL queries, JSON templates
- **Switch Expressions** - Use `->` syntax with yield when appropriate
- **Pattern Matching for `instanceof`** - Use `if (obj instanceof String s)` instead of casting
- **Pattern Matching for `switch`** - Use typed patterns in switch when applicable
- **Sealed classes** - Use for restricted hierarchies when applicable
- **String methods** - Use `.strip()`, `.isBlank()`, `.formatted()`, etc.
- **Virtual Threads** - Use when beneficial for I/O-bound concurrent operations

---

## 2. Architecture & Package Structure

Every microservice follows this package structure:

```
├── resources/              # REST endpoints (JAX-RS Resources)
├── service/                # Service interfaces
│   └── impl/               # Service implementations
├── repository/             # Panache repositories (data access + queries)
├── dto/                    # Data Transfer Objects (request/response)
├── entities/               # JPA entities
│   ├── enums/              # Enum types used by entities
│   └── converters/         # JPA attribute converters
├── exceptions/             # Custom exceptions (BusinessException, etc.)
│   └── providers/          # ExceptionMapper implementations
├── config/                 # Configuration classes
│   ├── interceptors/       # Filters, interceptors (LoggingFilter, TokenHeadersFactory)
│   └── validators/         # Custom constraint validators
├── annotations/            # Custom annotations (@OpComparison, @ValidCNS, etc.)
├── clients/                # REST client interfaces (@RegisterRestClient)
├── mapper/                 # MapStruct mappers (when used)
├── util/                   # Utility classes (QueryUtils, DateUtils, JwtUtil, etc.)
├── health/                 # Health check implementations
├── startup/                # Application startup hooks
└── concurrency/            # Interceptors and listeners for async operations
```

**Rules:**
- Separate files correctly into their packages/modules
- Do NOT mix concerns: a Service does not belong in `resources/`, a query does not belong in `service/`
- One class per file. Name the file exactly as the class name.

---

## 3. Resource Layer (Controllers)

Resources are thin HTTP controllers. They delegate ALL logic to Services.

```java
@AllArgsConstructor
@Authenticated
@Path("/v1/products")
public class ProductResources {


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flaviodotcom/quarkus-java-backend-playbook](https://github.com/flaviodotcom/quarkus-java-backend-playbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
