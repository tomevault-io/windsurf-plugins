---
trigger: always_on
description: Full-stack monorepo with:
---

# AGENTS.md — Codebase Guide for AI Coding Agents

## Project Overview

Full-stack monorepo with:
- **`order-api/`** — Spring Boot 4.0.6 REST API (Java 25, PostgreSQL, JWT auth)
- **`order-ui/`** — React 19 SPA (JavaScript, Vite 8, Axios, Mantine v9.1.1)

Authentication is stateless JWT (10-minute expiry, no refresh tokens). The backend uses domain-grouped packaging; the frontend uses feature-grouped folders.

---

## Build & Run Commands

### Backend (`order-api/`)

```bash
# Start the application (requires Postgres — see compose.yaml)
docker compose up -d               # start Postgres 18.3
./mvnw clean spring-boot:run       # run the API on :8080

# Build
./mvnw clean package               # build JAR with tests
./mvnw clean package -DskipTests   # build JAR without tests

# Run all tests
./mvnw test

# Run a single test class
./mvnw test -Dtest=OrderApiApplicationTests

# Run a single test method
./mvnw test -Dtest=OrderApiApplicationTests#contextLoads

# Run tests matching a pattern
./mvnw test -Dtest="*Controller*"
```

> **Note:** `OrderApiApplicationTests` uses `@MockitoBean` to mock all infrastructure and runs without a live Postgres. New tests should use `@WebMvcTest` for controllers, `@DataJpaTest` for repositories, and `@ExtendWith(MockitoExtension.class)` for service unit tests.

### Frontend (`order-ui/`)

```bash
# Install dependencies
npm install

# Start dev server on :3000
npm start

# Production build
npm run build

# Run all tests (CI mode — runs once and exits)
npm test

# Run a single test file by path pattern
npm test -- src/components/home/Login

# Run tests matching a name pattern
npm test -- -t "renders login"

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
> plus `eslint-plugin-react`, `eslint-plugin-react-hooks`, and `eslint-config-prettier`. Prettier is configured via `.prettierrc`.

> **Note:** Test files (`ComponentName.test.js` / `ComponentName.test.jsx`) are co-located with every component. New tests should use `@testing-library/react` + `@testing-library/user-event` (both already installed). `setupTests.js` registers `@testing-library/jest-dom` matchers via `expect.extend()` and mocks `matchMedia` and `localStorage` for Mantine compatibility.

### Integration Tests

```bash
# Manual integration test script (requires both services running)
./order-api/test-endpoints.sh      # zsh script; hits all endpoints with curl
```

---

## Architecture & Key Patterns

### Backend

- **Domain-grouped packages**, not layered: `order/`, `user/`, `security/`, `rest/`, `rest/dto`, `config/`, `runner/`
- **Four REST controllers**: `AuthController` (`/auth/**`), `OrderController` (`/api/orders/**`), `UserController` (`/api/users/**`), `PublicController` (`/public/**`)
- **DTOs are Java records**: `record LoginRequest(String username, String password) {}`
- **Entities use Lombok**: `@Data`, `@NoArgsConstructor` on JPA entities — no manual getters/setters
- **`@RequiredArgsConstructor`** on all Spring beans instead of `@Autowired`
- **`@ResponseStatus` on exceptions** instead of `@ControllerAdvice`:
  ```java
  @ResponseStatus(HttpStatus.NOT_FOUND)
  public class UserNotFoundException extends RuntimeException { ... }
  ```
  All domain exceptions follow this pattern:
  - `UserNotFoundException` (`404 NOT_FOUND`)
  - `DuplicatedUserInfoException` (`409 CONFLICT`)
  - `UserDeletionNotAllowedException` (`400 BAD_REQUEST`) — guards self-deletion and last-admin deletion
  - `OrderNotFoundException` (`404 NOT_FOUND`)
- **Service layer** is a simple class (`UserService`, `OrderService`) — no interface needed for single implementation
- **Optional<T>** used for nullable service lookups; `validateAndGet{Entity}By{Key}()` methods throw on empty
- **Ordered repository queries**: use Spring Data derived query methods for deterministic ordering — e.g., `findAllByOrderByUsernameAsc()` in `UserRepository`, `findAllByOrderByCreatedAtDesc()` in `OrderRepository`. Do not rely on `findAll()` where order matters.
- **JWT errors**: each exception type caught individually, logged with `@Slf4j`, returns `Optional.empty()`
- **`DatabaseInitializer`** (`CommandLineRunner` in `runner/`) seeds two default users on every startup: `admin`/`admin` (`ADMIN` role) and `user`/`user` (`USER` role). Because `ddl-auto: create` drops and recreates the schema on every boot, the user count is always 0 at startup, so seeding always runs.

### Frontend

- **Function components with hooks only** — no class components anywhere
- **Routing**: React Router v7 (`react-router-dom ^7`) with `<BrowserRouter>`, `<Routes>/<Route>`, and `<Navigate>`; unmatched paths redirect to `/`
- **UI library**: Mantine v9.1.1 (`@mantine/core`, `@mantine/hooks`) — requires `matchMedia` mock in tests (provided by `setupTests.js`)
- **Centralized API layer**: all Axios calls live in `src/components/misc/OrderApi.js`
- **Auth state** managed via React Context in `src/components/context/AuthContext.jsx` (use `useAuth()` hook)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ivangfr/springboot-react-jwt-token](https://github.com/ivangfr/springboot-react-jwt-token) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
