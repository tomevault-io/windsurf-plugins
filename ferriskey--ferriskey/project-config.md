---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

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

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ferriskey/ferriskey](https://github.com/ferriskey/ferriskey) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
