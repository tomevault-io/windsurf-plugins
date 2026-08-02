---
trigger: always_on
description: This file documents conventions, commands, and patterns for agentic coding agents working in this repository.
---

# AGENTS.md — Coding Agent Reference

This file documents conventions, commands, and patterns for agentic coding agents working in this repository.

---

## Project Overview

Full-stack social login application:
- **Backend:** `movie-api/` — Spring Boot 4.0.6, Java 25, PostgreSQL
- **Frontend:** `movie-ui/` — React 19, Vite 8 + Vitest 4, JavaScript (no TypeScript)
- **Infrastructure:** Docker Compose (PostgreSQL 18.3)

Default URLs: backend `http://localhost:8080`, frontend `http://localhost:3000`.
Swagger UI: `http://localhost:8080/swagger-ui.html`.

---

## Build, Run, and Test Commands

### Infrastructure

```bash
# Start PostgreSQL
docker compose up -d

# Stop and remove containers + volumes
docker compose down -v
```

### Backend (`movie-api/`)

All Maven commands must be run from the `movie-api/` directory using the wrapper.

```bash
# Run the application (requires running Postgres)
./mvnw clean spring-boot:run

# Run all tests
./mvnw test

# Build JAR (skip tests)
./mvnw clean package -DskipTests

# Run a single test class
./mvnw test -Dtest=MovieApiApplicationTests

# Run a single test method
./mvnw test -Dtest=MovieApiApplicationTests#contextLoads

# Run tests matching a pattern
./mvnw test -Dtest="*Movie*"
```

Integration smoke tests (requires a running app and `jq`):
```bash
./movie-api/test-endpoints.sh
```

### Frontend (`movie-ui/`)

All npm commands must be run from the `movie-ui/` directory.

```bash
# Install dependencies
npm install

# Start dev server (port 3000)
npm start

# Production build
npm run build

# Run all tests (non-interactive)
npm test

# Run tests for a specific file
npm test -- src/components/home/Login.test.jsx

# Run tests matching a name pattern
npm test -- --reporter=verbose Login

# Lint source files
npm run lint

# Lint and auto-fix safe issues
npm run lint -- --fix
```

> ESLint is configured via `eslint.config.js` (flat config) using `@eslint/js` recommended as a base,
> plus `eslint-plugin-react` and `eslint-plugin-react-hooks`. Prettier is available via `npm run format` / `npm run format:check` (no `.prettierrc` config file; uses Prettier defaults).

---

## Backend Code Style (Java)

### Formatting
- **Indentation:** 4 spaces (no tabs).
- **Braces:** K&R style — opening brace on the same line as declaration.
- **Blank lines:** Single blank line between methods; no trailing blank lines.
- Formatting is enforced via Spotless Maven plugin (v3.4.0) with Google Java Format (v1.35.0), running during `mvn verify`. No Checkstyle or SpotBugs configuration is enforced.

### Import Ordering
Group and order imports as follows (blank line between groups), enforced by Spotless:
1. `java.*`
2. `jakarta.*`
3. `org.*` (includes `org.springframework.*`, `org.projectlombok.*`, etc.)
4. Third-party libraries (`com.*` non-project, `io.*`)
5. Static imports last

### Naming Conventions

| Artifact | Convention | Example |
|---|---|---|
| JPA entities | `PascalCase`, no suffix | `User`, `Movie` |
| Repositories | `PascalCase` + `Repository` | `MovieRepository` |
| Services | `PascalCase` + `Service` | `MovieService`, `UserService` |
| REST controllers | domain + `Controller` | `MovieController` |
| Request DTOs (records) | action + domain + `Request` | `CreateMovieRequest` |
| Response DTOs (records) | domain + `Dto` / `Response` | `MovieDto`, `AuthResponse` |
| Custom exceptions | descriptive + `Exception` | `MovieNotFoundException` |
| Config classes | descriptive + `Config` | `SecurityConfig` |
| Enums | `PascalCase`; values `SCREAMING_SNAKE_CASE` | `OAuth2Provider.GITHUB` |
| Constants (fields) | `SCREAMING_SNAKE_CASE` | `TOKEN_TYPE`, `TOKEN_HEADER` |
| Instance fields / methods | `camelCase` | `jwtSecret`, `validateAndGetMovie` |
| Packages | lowercase, domain-scoped | `com.ivanfranchin.movieapi.security.oauth2` |

### Lombok Usage
- `@Data` + `@NoArgsConstructor` on JPA entities.
- `@RequiredArgsConstructor` on Spring beans (controllers, services, filters) — inject `final` fields; never use `@AllArgsConstructor` on beans.
- Exception: `CustomOAuth2UserService` uses a manual constructor (not `@RequiredArgsConstructor`) because Lombok cannot cleanly handle a `List<OAuth2UserInfoExtractor>` generic-collection dependency.
- `@Getter` + `@AllArgsConstructor(access = AccessLevel.PRIVATE)` on immutable non-bean classes (e.g. `CustomUserDetails`); expose construction via named static factory methods, never via the raw constructor.
- `@Slf4j` on any class that logs.

### DTOs
- All request and response DTOs are **Java records** (immutable).
- Use component accessor syntax: `loginRequest.username()` (not getters).
- Static factory: `public static Dto from(DomainObject o)`. Multiple `from()` overloads are allowed when the same DTO can be built from different source types (e.g. `UserDto.from(User)` and `UserDto.from(CustomUserDetails)`).
- Domain mapping: `public DomainObject toDomain()` instance method on the record.

### Package Structure
- Domain packages co-locate entity, repository, service, and exceptions: `movie/`, `user/`.
- REST layer is in `rest/` with a `dto/` sub-package.
- Cross-cutting concerns live in `security/`, `config/`, `runner/`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ivangfr/springboot-react-social-login](https://github.com/ivangfr/springboot-react-social-login) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
