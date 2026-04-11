---
trigger: always_on
description: This file provides architectural guidelines and development standards for Gemini when working in this repository (optimized for Zed Assistant).
---

# GEMINI.md - Context & Guidance for FerrisKey

This file provides architectural guidelines and development standards for Gemini when working in this repository (optimized for Zed Assistant).

## Overview

FerrisKey is a modern, open-source Identity & Access Management (IAM) system built in Rust with a hexagonal architecture. It consists of:
- **core** - Business logic and domain entities (hexagonal architecture)
- **api** - HTTP API layer built with Axum
- **operator** - Kubernetes operator
- **front** - React/TypeScript frontend (Vite, TailwindCSS, Radix UI)

## Development Commands

### Backend (Rust)

```bash
# Database setup (required first time)
cd core
DATABASE_URL=postgres://ferriskey:ferriskey@localhost:5432/ferriskey sqlx migrate run

# Run API server (from api/)
cd ../api
cargo run

# Run all tests
cargo test

# Run tests for specific package
cargo test -p ferriskey-core
cargo test -p ferriskey-api

# Run integration tests (requires test database)
cargo test --test it

# Check code (faster than build)
cargo check

# Format code
cargo fmt

# Lint
cargo clippy
```

### Frontend (React)

```bash
cd front

# Install dependencies
pnpm install

# Dev server (port 5555)
pnpm run dev

# Build for production
pnpm run build

# Lint
pnpm run lint

# Preview production build
pnpm run preview
```

### Docker

```bash
# Full stack with local build
docker compose --profile local up -d

# Full stack with registry image
docker compose --profile registry up -d
```

## Architecture

### Hexagonal Architecture (Ports & Adapters)

The `core` crate strictly follows hexagonal architecture:

```
core/src/
├── domain/           # Pure business logic (no infrastructure)
│   ├── authentication/
│   ├── user/
│   ├── client/
│   ├── realm/
│   ├── credential/
│   ├── role/
│   ├── trident/      # MFA system
│   ├── seawatch/     # Audit logging
│   ├── webhook/
│   ├── jwt/
│   └── common/       # Shared domain types
├── application/      # Application services (orchestration)
├── infrastructure/   # Concrete implementations
│   ├── repositories/ # Database access via SeaORM
│   └── db/postgres/  # Connection & migrations
└── entity/          # SeaORM auto-generated entities
```

**Key Pattern:**
Each domain module contains:
- `entities.rs` - Domain value objects (immutable)
- `ports.rs` - Trait definitions (repository/service contracts)
- `services.rs` - Business logic implementation
- `value_objects.rs` - DTOs for use cases
- `policies.rs` - Authorization policies (where applicable)

**Dependency Flow:**
- Domain depends on nothing
- Application depends on domain (uses ports)
- Infrastructure implements ports
- `ApplicationService` in `application/mod.rs` composes everything via dependency injection

### API Layer (`api/src/application/http/`)

Built with Axum. Each feature mirrors a domain module:

```
http/
├── server/              # HTTP setup, routing, middleware
│   ├── http_server.rs  # Main router composition
│   ├── app_state.rs    # Application state (contains services)
│   └── openapi.rs      # Swagger/ReDoc/Scalar
├── authentication/      # Auth endpoints
├── user/
├── client/
├── realm/
├── role/
├── trident/            # MFA endpoints
├── seawatch/           # Security events
└── webhook/
```

**Handler Pattern:**
- Each feature has `router.rs` (routes), `handlers/` (endpoint logic), `validators.rs`, `errors.rs`
- Handlers extract `State<AppState>` to access domain services
- Errors convert: `CoreError` → `ApiError` (see `api/src/application/http/errors/error.rs`)
- OpenAPI docs via `utoipa` attributes on handlers

### Frontend (`front/`)

React 19 + TypeScript + Vite stack:

```
src/
├── api/                 # API client layer
│   ├── api.client.ts   # Auto-generated from OpenAPI
│   ├── api.tanstack.ts # React Query wrappers
│   └── *.api.ts        # Feature-specific calls
├── pages/              # Feature pages (match backend modules)
├── components/         # Reusable UI (Radix UI primitives)
├── hooks/              # Custom hooks
├── store/              # Zustand state (auth, user, realm)
├── types/              # TypeScript definitions
└── routes/             # React Router v7
```

**State Management:**
- **Server state:** React Query (TanStack Query)
- **Client state:** Zustand (auth, realm context)
- **Forms:** react-hook-form + Zod validation
- **Frontend API calls:** Prefer the generated `createApiClient` / `window.tanstackApi` client and TanStack Query hooks or mutations over direct `axios` calls. Use `axios` only when a flow cannot be expressed through the generated client.

## Database & Migrations

- **PostgreSQL** via SeaORM (async ORM)
- Migrations in `core/migrations/` (timestamped SQL files)
- Entities auto-generated in `core/src/entity/` (do not edit manually)

**Running migrations:**
```bash
cd core
DATABASE_URL=postgres://ferriskey:ferriskey@localhost:5432/ferriskey sqlx migrate run
```

**Generating entities after migration:**
```bash
# Install sea-orm-cli if not present
cargo install sea-orm-cli

# Generate from database
sea-orm-cli generate entity \
  --database-url postgres://ferriskey:ferriskey@localhost:5432/ferriskey \
  --output-dir src/entity
```

## Testing Strategy

**Unit Tests:**
- Embedded in source files with `#[test]` or `#[tokio::test]`
- Domain services use `mockall` for repository mocking
- Infrastructure tests validate implementations (e.g., Argon2, recovery codes)

**Integration Tests:**
- Located in `api/tests/it/`
- Use `test-context` crate for test lifecycle
- `postgres_context.rs` helper for database setup
- `axum-test` for HTTP endpoint testing

**Run specific test:**
```bash
# Single test function
cargo test test_name

# All tests in a file
cargo test --test it

# With output
cargo test -- --nocapture
```

## Key Domain Concepts

**Multi-Tenancy (Realms):**
- Every user, client, role, credential belongs to a realm
- Complete isolation between realms
- Realm-specific configuration and settings

**Bitwise Role System:**
- Roles are bitmasks for efficient permission checks
- User/client roles stored as integers
- Quick AND operations for authorization

**Required Actions:**
- Users can have required actions (e.g., MFA setup, password reset)
- Enforced before full authentication completes
- Handled in `domain/user/required_actions.rs`

**Trident Module (MFA):**
- TOTP (Time-based One-Time Password)
- WebAuthn (passwordless with security keys)
- Magic Links (email-based)
- Recovery Codes (backup codes)

**SeaWatch Module (Audit):**
- Security event logging for all critical actions
- Queryable event trail
- Exportable audit logs

**Webhooks:**
- Event-driven extensibility
- Subscribe to lifecycle events (user created, client updated, etc.)
- Async delivery to configured endpoints

## Error Handling

Layered error types:
1. `CoreError` (domain) - Business logic errors in `core/src/domain/common/errors.rs`
2. `ApiError` (HTTP) - HTTP response format in `api/src/application/http/errors/error.rs`
3. Automatic conversion via `From<CoreError> for ApiError`

Errors are traced through the system with context.

## Configuration

- **Clap** for CLI args and environment variables
- Main config in `api/src/Args` struct
- Environment variables override defaults
- Copy `.env.example` to `.env` in `api/` directory

**Key variables:**
```bash
DATABASE_URL=postgres://ferriskey:ferriskey@localhost:5432/ferriskey
PORT=3333
ADMIN_USERNAME=admin
ADMIN_PASSWORD=admin
ADMIN_EMAIL=admin@ferriskey.rs
ALLOWED_ORIGINS=http://localhost:5555
LOG_LEVEL=info
```

## Observability

- **Metrics:** Prometheus endpoint at `/metrics` (via `axum-prometheus`)
- **Tracing:** Structured logging with `tracing` crate
- **OpenTelemetry:** Optional OTLP integration for distributed tracing
- **OpenAPI Docs:** Available at `/swagger-ui`, `/redoc`, `/rapidoc`, `/scalar`

## Common Patterns

**Adding a new domain entity:**
1. Create migration in `core/migrations/`
2. Run migration and regenerate entities
3. Create domain module in `core/src/domain/your_feature/`
4. Define entities, ports, services, value_objects
5. Implement repository in `core/src/infrastructure/repositories/`
6. Wire up in `ApplicationService` (`core/src/application/mod.rs`)
7. Add HTTP handlers in `api/src/application/http/your_feature/`
8. Add routes to main router in `api/src/application/http/server/http_server.rs`

**Repository pattern:**
All repositories are traits defined in `domain/*/ports.rs`, implemented in `infrastructure/repositories/*_repository.rs`. Services depend on traits, not concrete implementations (enables mocking).

**Generic services:**
Domain services use generic type parameters for repositories (e.g., `UserService<R: UserRepository>`). Type aliases in `application/services.rs` provide concrete types.

**Error propagation:**
Use `?` operator liberally. Errors convert automatically via `From` implementations. Add context with `.map_err()` where helpful.

## CI/CD

GitHub Actions workflows in `.github/workflows/`:
- `docker.yaml` - Builds and pushes Docker images on main/tags
- `helm.yaml` - Packages and publishes Helm charts
- `pre-commit.yaml` - Runs pre-commit hooks
- `release.yaml` - Creates GitHub releases

## Important Notes

- **Never edit** `core/src/entity/` files manually - they're generated from migrations
- **Async everywhere** - Use `tokio::test` for async tests, `.await` on all DB/HTTP calls
- **Type safety** - Leverage Rust's type system; prefer compile-time errors over runtime
- **No unwrap()** - Current refactor (branch 623-remove-unwrap-in-codebase) eliminates `.unwrap()` calls
- **Cookie handling** - Uses `axum-extra` for cookie management (see recent commit 1fc8d81)
- **Default credentials** - admin/admin for local development (change in production)

## Useful Links

- Documentation: https://docs.ferriskey.rs/getting-started/introduction
- Helm Chart: `oci://ghcr.io/ferriskey/charts/ferriskey`
- Discussions: https://github.com/ferriskey/ferriskey/discussions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ferriskey)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ferriskey)
<!-- tomevault:4.0:windsurf_rules:2026-04-08 -->
