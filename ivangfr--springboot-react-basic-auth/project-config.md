---
trigger: always_on
description: Full-stack application with:
---

# AGENTS.md — Codebase Guide for AI Coding Agents

## Project Overview

Full-stack application with:
- **Backend**: `book-api/` — Spring Boot 4.0.6, Java 25, PostgreSQL, HTTP Basic Auth, stateless REST API
- **Frontend**: `book-ui/` — React 19, plain JavaScript (no TypeScript), Axios, Mantine v9, @mantine/hooks

---

## Build, Lint, and Test Commands

### Backend (`book-api/`)

```bash
# Run the application (requires PostgreSQL running via docker compose)
./mvnw clean spring-boot:run

# Build a JAR
./mvnw clean package

# Run all tests
./mvnw test

# Run a single test class
./mvnw test -Dtest=BookApiApplicationTests

# Run a single test method
./mvnw test -Dtest=BookApiApplicationTests#contextLoads

# Run tests matching a pattern
./mvnw test -Dtest="BookApi*"

# Manual integration test (requires running app + DB)
./book-api/test-endpoints.sh
```

> Spotless is configured with `google-java-format` (GOOGLE style) for code formatting. Run `./mvnw spotless:check` to verify formatting, `./mvnw spotless:apply` to auto-fix. Spotless runs automatically during `./mvnw verify`. No Checkstyle plugin is configured.

### Frontend (`book-ui/`)

```bash
# Install dependencies
npm install

# Start dev server (http://localhost:3000)
npm start

# Production build
npm run build

# Run all tests (non-interactive, single pass)
npm test

# Run a single test file
npm test -- src/components/admin/AdminPage.test.jsx

# Run tests matching a name pattern
npm test -- -t "test description"

# Lint source files
npm run lint

# Lint and auto-fix safe issues
npm run lint -- --fix

# Format source files
npm run format

# Check formatting
npm run format:check
```

> ESLint is configured via `eslint.config.js` (flat config) using `@eslint/js` recommended as a base,
> plus `eslint-plugin-react`, `eslint-plugin-react-hooks`, and `eslint-config-prettier`. Prettier is configured with `format` and `format:check` scripts.

### Infrastructure

```bash
# Start PostgreSQL (required before running backend)
docker compose up -d
```

---

## Code Style Guidelines

### Java (Backend)

- **Java version**: 25. Use modern Java features appropriately.
- **Indentation**: 4 spaces.
- **DTOs**: Use Java `record` types for all request and response DTOs. Never use mutable classes for DTOs.
- **Entities**: Plain classes with Lombok annotations. `@Data` and `@NoArgsConstructor` are used on all entities; `@AllArgsConstructor` is added when all fields are needed in the constructor (e.g., `Book`). Entities that require a partial or custom constructor (e.g., `User`) use a hand-written constructor instead of `@AllArgsConstructor`. Use `@Slf4j` for logging where needed (e.g., `DatabaseInitializer`).
- **No service interfaces**: Services are concrete classes (e.g., `BookService.java`, `UserService.java`). No `XxxService` interface + `XxxServiceImpl` pattern.
- **No `var`**: Avoid `var`; prefer explicit types for clarity.
- **Import ordering** (blank line between each group, enforced by Spotless):
  1. Java standard library (`java.*`)
  2. Third-party libraries (`jakarta.*`, `lombok.*`, `io.swagger.*`)
  3. Spring Framework and other `org.*` libraries (`org.springframework.*`)
  4. Project-internal (`com.ivanfranchin.*`)
  5. Static imports last (after a blank line)
- **Formatting**: Enforced by Spotless using `google-java-format` (GOOGLE style). Removes unused imports, forbids wildcard imports. Run `./mvnw spotless:apply` to auto-format.

- **Validation**: Use Bean Validation annotations (`@NotBlank`, `@Email`) on record components and activate with `@Valid` on controller parameters.
- **Controllers**: Return plain domain objects or records where possible. Use `ResponseEntity` only when the HTTP status must be conditional at runtime.

### JavaScript (Frontend)

- **Language**: Plain JavaScript (ES2020+). No TypeScript.
- **File extensions**: React components use `.jsx`; non-component JS files (utilities, constants, config) use `.js`.
- **Indentation**: 2 spaces.
- **Quotes**: Single quotes for all strings and imports.
- **Semicolons**: None — the codebase uses a no-semicolon style consistently.
- **Components**: Functional components only. No class components.
- **Import ordering** (no blank-line separation enforced, but follow this order):
  1. React and react-* packages (`react`, `react-router-dom`)
  2. Third-party UI library (`@mantine/core`, `@tabler/icons-react`)
  3. Internal context (`../context/AuthContext`)
  4. Internal services/utilities (`../misc/BookApi`, `../misc/Helpers`)
  5. Sibling components
- **No Prettier config** — match the existing style of the file being edited.

---

## Naming Conventions

### Java

| Artifact | Convention | Examples |
|---|---|---|
| Classes | `PascalCase` + role suffix | `BookController`, `BookServiceImpl`, `DatabaseInitializer` |
| Interfaces | `PascalCase`, no prefix/suffix | `BookService`, `UserService` |
| Records (DTOs) | `PascalCase` + `Request`/`Response`/`Dto` | `CreateBookRequest`, `BookDto`, `AuthResponse` |
| Exceptions | `PascalCase` + `Exception` | `BookNotFoundException`, `DuplicatedUserInfoException`, `UserDeletionNotAllowedException`, `UserNotFoundException` |
| Methods | `camelCase`, verb-first | `getBooks`, `validateAndGetBook`, `hasUserWithUsername` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ivangfr/springboot-react-basic-auth](https://github.com/ivangfr/springboot-react-basic-auth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
