---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Educational Spring Boot 3.5.6 REST API for user management with comprehensive JavaDoc documentation in Spanish. Demonstrates modern Java development practices including three-layer architecture, Bean Validation, transactional operations, and OpenAPI integration. Uses Java 17 with Gradle build system and H2 in-memory database.

**Target Audience:** Computer science students learning Spring Boot architecture and best practices.

## Build & Run Commands

**Note:** On Windows, use `gradlew.bat` instead of `./gradlew` for all commands below.

**Run the application (production mode):**
```bash
./gradlew bootRun
# Windows: gradlew.bat bootRun
```

**Run with development profile (enables SQL logs and H2 console):**
```bash
./gradlew bootRun --args='--spring.profiles.active=dev'
# Windows: gradlew.bat bootRun --args="--spring.profiles.active=dev"
```

Application runs on `http://localhost:8080`

**Build the project:**
```bash
./gradlew build
```

**Build executable JAR:**
```bash
./gradlew bootJar
```
JAR location: `build/libs/usuarios-api.jar`

**Run JAR:**
```bash
java -jar build/libs/usuarios-api.jar
```

**Run JAR with dev profile:**
```bash
java -jar build/libs/usuarios-api.jar --spring.profiles.active=dev
```

**Run tests:**
```bash
./gradlew test
```

**Clean build:**
```bash
./gradlew clean build
```

## Architecture

**Three-Layer Architecture:**
- **Controller Layer** (`controller/`): REST endpoints at `/api/usuarios` using `ResponseEntity<>` for explicit HTTP status control (201 CREATED, 200 OK, 204 NO CONTENT)
- **Service Layer** (`service/`): Business logic with `@Transactional` on ALL write operations (crearUsuario, actualizarUsuario, eliminarUsuario). Validates entity existence before updates/deletes to prevent race conditions
- **Repository Layer** (`repository/`): Data access using Spring Data JPA

**DTO Pattern:**
- `UsuarioRequestDTO`: Input validation with Jakarta Bean Validation annotations (@NotBlank, @Email, @Size). Used for both create and update operations.
- `UsuarioResponseDTO`: Output mapping, excludes sensitive fields (password). Static factory method `fromEntity()` converts entities to DTOs.
- DTOs separate internal entity representation from API contracts

**Global Exception Handling:**
`GlobalExceptionHandler` uses `@ControllerAdvice` to centralize error handling with standardized `ErrorDTO` responses:
- `MethodArgumentNotValidException`: Bean validation errors (HTTP 400)
- `IllegalArgumentException`: Business logic validation errors like missing users (HTTP 400)
- `ConstraintViolationException`: Database constraint violations (HTTP 400)
- `DataIntegrityViolationException`: Duplicate email detected via `getRootCause()` analysis (HTTP 409 CONFLICT)
- Generic exceptions return HTTP 500 with sanitized messages (no stack trace exposure in production)

**Database Configuration:**
H2 in-memory database configured in `application.properties`:
- URL: `jdbc:h2:mem:usuariosdb`
- H2 Console: `http://localhost:8080/h2-console` (disabled by default, enable with `H2_CONSOLE_ENABLED=true`)
- JPA auto-creates schema on startup (`spring.jpa.hibernate.ddl-auto=update`)
- SQL logging disabled by default (enable with `JPA_SHOW_SQL=true` or use dev profile)
- Entity constraints: All fields are `nullable=false`, email has `unique=true`, lengths defined (nombre: 100, email/password: 255)

## API Documentation

OpenAPI/Swagger UI configured via SpringDoc:
- Access at: `http://localhost:8080/swagger-ui.html`
- Configuration in `config/OpenApiConfig.java`

## API Endpoints

Base path: `/api/usuarios`

- `POST /api/usuarios` - Create user (returns **201 CREATED**, validates input, returns 409 if email exists)
- `GET /api/usuarios` - List all users (returns **200 OK**)
- `GET /api/usuarios/{id}` - Get user by ID (returns **200 OK** or 400 if not found)
- `PUT /api/usuarios/{id}` - Update user (returns **200 OK**, validates input, returns 400 if not found, 409 if email conflicts)
- `DELETE /api/usuarios/{id}` - Delete user (returns **204 NO CONTENT** or 400 if not found)

## JavaDoc Documentation

All source files contain **extensive educational JavaDoc in Spanish** explaining:
- Architecture concepts (three-layer pattern, dependency injection)
- Design patterns (Builder, Factory Method, DTO)
- Spring annotations (@Service, @Transactional, @Valid, @ControllerAdvice)
- Transaction management and race condition prevention
- Optional API and Streams usage
- HTTP status code semantics

**Generate HTML documentation:**
```bash
./gradlew javadoc
```
Output: `build/docs/javadoc/index.html`

**JavaDoc Configuration:** Configured in `build.gradle` with `Xdoclint:none` to allow educational formatting (tables, code examples, multiple heading levels).

## Key Implementation Details

### Transactional Boundaries & Race Condition Prevention

ALL write operations use `@Transactional` to ensure atomicity:
- `crearUsuario()`: Guarantees atomic construction, persistence, and DTO conversion
- `actualizarUsuario()`: Ensures findById + modify + save execute as atomic unit
- `eliminarUsuario()`: Prevents TOCTOU race conditions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cortezalberto) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
