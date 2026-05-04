---
trigger: always_on
description: - **Always use the latest LTS version of Java** and leverage the newest language features available (e.g., records, sealed classes, pattern matching, virtual threads, string templates).
---

# Agent Instructions

## Language & Frameworks

### Java
- **Always use the latest LTS version of Java** and leverage the newest language features available (e.g., records, sealed classes, pattern matching, virtual threads, string templates).
- **Lombok is mandatory.** Use Lombok annotations to reduce boilerplate:
    - `@Data`, `@Value`, `@Builder`, `@AllArgsConstructor`, `@NoArgsConstructor`, `@Slf4j`, `@RequiredArgsConstructor`, etc.
    - Never write getters, setters, `toString()`, `equals()`, or `hashCode()` manually when Lombok can generate them.
- When using **Spring**, always use **Spring Boot 4** (Spring Framework 7). Do not use older versions.
    - Prefer constructor injection (via `@RequiredArgsConstructor`) over field injection.
    - Use `application.yml` over `application.properties`.

## Architecture & Code Organization

### Domain-Driven Design (DDD)

- **Always apply Domain-Driven Design principles** when organizing code into Java classes and packages.
- Structure the codebase around **bounded contexts** and **domain modules**, not technical layers.
- Preferred package structure per domain/module:

  ```
  com.example.project
  ├── order/                        # Bounded context: Order
  │   ├── domain/
  │   │   ├── model/                # Entities, Value Objects, Aggregates
  │   │   ├── event/                # Domain Events
  │   │   ├── repository/           # Repository interfaces (ports)
  │   │   └── service/              # Domain services (pure business logic)
  │   ├── application/              # Application services / use cases
  │   ├── infrastructure/
  │   │   ├── persistence/          # Repository implementations (adapters)
  │   │   ├── messaging/            # Event publishers, message consumers
  │   │   └── client/               # External service clients
  │   └── api/                      # REST controllers, DTOs, mappers
  │       ├── dto/
  │       └── mapper/
  ├── customer/                     # Bounded context: Customer
  │   ├── domain/
  │   ├── application/
  │   ├── infrastructure/
  │   └── api/
  └── shared/                       # Shared kernel (cross-cutting concerns)
      ├── domain/
      └── infrastructure/
  ```

- **Entities** must have a clear identity and lifecycle. Use JPA `@Entity` with Lombok `@Data` or `@Getter`/`@Setter`.
- **Value Objects** must be immutable. Use Java `record` types or Lombok `@Value`.
- **Aggregates** must enforce consistency boundaries. Only the aggregate root should be accessible from outside the aggregate.
- **Domain Services** contain business logic that doesn't naturally belong to a single entity.
- **Application Services** orchestrate use cases by coordinating domain objects and infrastructure.
- **Repository interfaces** belong in the domain layer; their implementations belong in infrastructure.
- Keep the domain layer **free of framework dependencies** (no Spring annotations, no JPA in model classes if possible — use mapping in the infrastructure layer).

### Consistency with Existing Patterns

- **When creating new functionalities, always analyze and follow the existing package-naming conventions and class-organization patterns** already present in the codebase.
- Before creating new packages or classes:
    1. Review how existing features are structured.
    2. Mirror the same naming conventions (e.g., `*Service`, `*Repository`, `*Controller`, `*Dto`, `*Mapper`).
    3. Place new classes in the same relative locations as their existing counterparts.
    4. Follow the same layering approach already established.
- **Never introduce a new organizational pattern** unless the existing one is explicitly being refactored. Consistency across the codebase takes priority.
- If the codebase does not yet follow DDD, propose a migration path in the documentation rather than mixing paradigms.

## Security

- **Sensitive data must always be encrypted.** Never store secrets, passwords, API keys, or tokens in plain text.
    - Use environment variables or a secrets manager (e.g., HashiCorp Vault, AWS Secrets Manager) for runtime secrets.
    - In Spring, use `jasypt-spring-boot` or Spring Cloud Vault for encrypted properties.
    - Database credentials, JWT secrets, and API keys must never appear in source code or configuration files committed to the repository.
- Always apply input validation and sanitization.
- Use parameterized queries or JPA/Hibernate — never concatenate SQL strings.
- Enable CSRF protection and CORS policies where applicable.
- Use HTTPS for all external communication.
- Apply the principle of least privilege for all service accounts and database users.

## Deployment & Containerization

- **All artifacts must be containerized with Docker.**
    - Every deployable service must include a `Dockerfile`.
    - Use multi-stage builds to keep images small and secure.
    - Base images should be minimal (e.g., `eclipse-temurin:<version>-jre-alpine`).
    - Never run containers as root — always define a non-root `USER` in the Dockerfile.
    - Use `.dockerignore` to exclude unnecessary files.

### Example Dockerfile

```dockerfile
# Build stage
FROM eclipse-temurin:23-jdk-alpine AS build

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tmseidel/restic-explorer](https://github.com/tmseidel/restic-explorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
