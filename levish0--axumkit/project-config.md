---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AxumKit is a production-ready Rust web API template built on Axum, SeaORM, PostgreSQL, Redis, and OAuth2. It implements JWT-based authentication with session management, OAuth2 integrations (Google, GitHub), and Cloudflare R2 storage.

## Common Commands

### Development
```bash
cargo run                          # Start the development server (localhost:8000)
cargo build                        # Build the project
cargo test                         # Run all tests
```

### Database Migrations
```bash
cd migration
cargo run                          # Apply all pending migrations
cargo run -- up                    # Same as above
cargo run -- down                  # Rollback last migration
cargo run -- fresh                 # Drop all tables and reapply migrations
cargo run -- refresh               # Rollback all, then reapply all migrations
cargo run -- status                # Check migration status
cargo run -- generate <NAME>       # Generate new migration file
```

### API Documentation
- Swagger UI: http://localhost:8000/docs (debug builds only)
- OpenAPI JSON: http://localhost:8000/swagger.json (debug builds only)

## Architecture

The project follows a layered architecture with clear separation of concerns:

```
API Layer (src/api/)
    ↓
Service Layer (src/service/)
    ↓
Repository Layer (src/repository/)
    ↓
Entity Layer (src/entity/)
```

### Key Components

**AppState** (`src/state.rs`): Application-wide shared state containing:
- `conn`: PostgreSQL database connection (SeaORM)
- `redis_client`: Redis connection manager for sessions/caching
- `http_client`: reqwest HTTP client for OAuth2 and external APIs

**Configuration** (`src/config/db_config.rs`):
- Centralized config using `LazyLock` for environment variables
- Access via `DbConfig::get()` - returns static reference
- Loaded from `.env` file on first access

**Error Handling** (`src/errors/`):
- Centralized error system with `Errors` enum
- Domain-specific error handlers (user, oauth, session, password, etc.)
- Errors automatically convert to HTTP responses via `IntoResponse`
- Development mode shows detailed error info; production mode hides it
- Standard result types: `ServiceResult<T>` and `ApiResult<T>`

**Authentication Flow**:
1. Session-based auth using Redis (not JWT tokens in production)
2. Session data stored as `session:{session_id}` in Redis with TTL
3. `session_auth` middleware extracts session from cookies and validates via Redis
4. `SessionContext` (containing `user_id` and `session_id`) injected into request extensions
5. Handlers extract `SessionContext` from request to get authenticated user info
6. OAuth2 providers (Google, GitHub) create sessions after successful authentication

**Middleware** (`src/middleware/`):
- `session_auth`: Validates session cookie and loads `SessionContext`
- `anonymous_user_middleware`: Handles unauthenticated requests
- `cors_layer`: CORS configuration from environment

**DTOs** (`src/dto/`):
- Organized by domain: `auth/`, `oauth/`, `user/`
- Each domain has `request/`, `response/`, and `internal/` subdirectories
- Internal DTOs (e.g., `Session`, `SessionContext`) not exposed in API

**Services** (`src/service/`):
- Business logic layer
- `SessionService`: Create/get/delete/refresh Redis sessions
- `auth/`: Login, logout, session management
- `oauth/`: OAuth2 flow handling for Google and GitHub
- `validator/`: Request validation (form and JSON)

**Repositories** (`src/repository/`):
- Database query layer
- Pattern: `find_by_*` returns `Option<Model>`, `get_by_*` returns `Result<Model, Errors>`
- Organized by domain (user, oauth)

### Route Structure

Routes are versioned and organized by domain:
```
/v0/health/*        - Health check endpoints
/v0/auth/*          - Authentication endpoints (login, logout, OAuth)
/docs               - Swagger UI (debug builds only)
/swagger.json       - OpenAPI spec (debug builds only)
```

OpenAPI documentation is auto-generated using `utoipa`:
- Define schemas with `#[derive(ToSchema)]`
- Document endpoints with `#[utoipa::path(...)]`
- Register in `src/api/v0/routes/openapi.rs`

## Environment Configuration

Required environment variables (see `.env.example`):
- `ENVIRONMENT`: Set to "dev" or "development" for development mode
- `JWT_SECRET`: Secret key for JWT signing
- `POSTGRES_*`: PostgreSQL connection settings
- `REDIS_HOST`, `REDIS_PORT`: Redis connection
- `HOST`, `PORT`: Server bind address
- `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, `GOOGLE_REDIRECT_URI`: Google OAuth
- `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET`, `GITHUB_REDIRECT_URI`: GitHub OAuth
- `R2_*`: Cloudflare R2 storage credentials
- `CORS_ALLOWED_ORIGINS`, `CORS_ALLOWED_HEADERS`: CORS configuration

## Development Patterns

### Adding New Endpoints

1. Create handler in `src/api/v0/routes/{domain}/{handler}.rs`
2. Add `#[utoipa::path(...)]` annotation for OpenAPI docs
3. Register route in `src/api/v0/routes/{domain}/routes.rs`
4. Add schemas to `src/api/v0/routes/{domain}/openapi.rs`
5. Create DTOs in `src/dto/{domain}/request/` and `response/`
6. Implement service logic in `src/service/{domain}/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [levish0/AxumKit](https://github.com/levish0/AxumKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
