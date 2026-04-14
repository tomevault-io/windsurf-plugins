---
trigger: always_on
description: <!-- Repository-specific guidance for AI coding agents. Keep concise and actionable. -->
---

<!-- Repository-specific guidance for AI coding agents. Keep concise and actionable. -->
# Copilot instructions for dispo-rusty (backend)

## Purpose
Help AI agents be immediately productive in this multi-tenant Actix Web + Diesel Rust backend with advanced functional programming patterns and Keycloak OAuth2 integration.

## Architecture & Core Concepts
- **Multi-tenant Isolation**: Single PostgreSQL database using per-tenant schemas. Tenant context is ALWAYS derived server-side (host/subdomain, mTLS, or lookup). NEVER trust client-supplied tenant IDs.
- **Authentication**: Dual auth system - JWT tokens (direct login) and optional Keycloak OAuth2 flows. Keycloak is not mandatory and can be disabled for pure JWT deployments. Three supported modes:
  - **Keycloak-required mode** (`AUTH_MODE=keycloak`): SSO deployments requiring Keycloak OAuth2. Server fails fast if `KEYCLOAK_ISSUER_URL`, `KEYCLOAK_CLIENT_ID`, or `KEYCLOAK_CLIENT_SECRET` are missing or invalid.
  - **JWT-only fallback mode** (`AUTH_MODE=jwt`): Pure JWT authentication without Keycloak. Server starts normally even if Keycloak variables are missing.
  - **Hybrid mode** (`AUTH_MODE=hybrid`): Supports both JWT and Keycloak OAuth2 flows. Server validates Keycloak settings at startup when Keycloak config is present.
  Server startup validates Keycloak settings only when `AUTH_MODE` includes keycloak, logging clear errors on misconfiguration. In JWT-only mode, Keycloak variables are ignored.
- **Functional Programming**: Leverages `rcs-functional` library for pure functions, iterator chains (`ChainBuilder`), and immutable state management.
- **NFAg Domain**: Implements Nota Fiscal Eletrônica da Agua (NFAg) with complex XSD-based validation and multi-tenant CRUD operations.
- **Real-time Observability**: Logs streamed over WebSocket at `/api/ws/logs` using `rcs::utils::ws_logger`.
- **WebSocket Logging**: Configurable buffer size (`WS_LOG_BUFFER_SIZE`) for real-time log streaming.

## Key Files & Directories
- **Entry Points**: [src/main.rs](src/main.rs), [src/lib.rs](src/lib.rs).
- **Functional Core**: [functional_lib/src/lib.rs](functional_lib/src/lib.rs) and `src/functional/` (re-exported as `functional`).
- **API Controllers**: [src/api/](src/api/) (e.g., [nfag_controller.rs](src/api/nfag_controller.rs)).
- **Configuration**: [src/config/app.rs](src/config/app.rs) (route registration), [src/config/db.rs](src/config/db.rs) (tenant pool manager).
- **Middleware**: [src/middleware/auth_middleware.rs](src/middleware/auth_middleware.rs).
- **Keycloak Integration**: [src/utils/keycloak.rs](src/utils/keycloak.rs) for OAuth2 flows.
- **Docker Setups**: `docker-compose.local.yml` (dev), `docker-compose.prod.yml` (production), `docker-compose.kafka.yml` (with Kafka).

## Conventions & Patterns
- **Route Building**: Use `RouteBuilder` in [src/config/app.rs](src/config/app.rs) for composable route registration.
- **Database Access**: Use `TenantPoolManager` to obtain per-tenant connection pools: `manager.get_pool(&tenant_id)?`.
- **Functional Logic**: Prefer `PureFunctionRegistry` and `ChainBuilder` over ad-hoc imperative logic.
- **Validation**: Use `ValidationEngine` for iterator-based validation pipelines, especially for NFAg XSD compliance.
- **Backward Compatibility**: Use `BackwardCompatibilityValidator` in [src/api/health_controller.rs](src/api/health_controller.rs) to ensure functional enhancements don't break existing APIs.
- **Pagination**: Use `unified_pagination` with encrypted cursors (AES-256-GCM).
- **Error Handling**: ServiceError with `.with_detail()` and `.with_tag()` for structured error responses.
- **OAuth2 Flow**: Keycloak redirect at `/api/auth/login/keycloak`, callback at `/api/callback`.

## Developer Workflows
- **Local Dev**: `cargo run` (reads `.env`).
- **Migrations**: `diesel migration generate <name>` then `diesel migration run`.
- **Testing**:
  - Unit: `cargo test`.
  - Integration: Requires Docker. Run with `cargo test --test integration_tests`.
  - Log Streaming: Run with `cargo test -- --test-threads=1` to avoid race conditions.
  - Functional Tests: Comprehensive tests in `tests/functional_tests.rs` covering iterator engines, pure functions, validation pipelines.
- **Code Quality**: `cargo clippy` and `cargo fmt`.
- **Docker**: `make docker-up-local` (dev stack), `make docker-up-prod` (prod stack), `make docker-up-kafka` (with Kafka).
- **Keycloak Setup**: Configure `KEYCLOAK_ISSUER_URL`, `KEYCLOAK_CLIENT_ID`, `KEYCLOAK_CLIENT_SECRET` in `.env`.

## Code Edit Guidelines
- **Preserve Tenant Isolation**: Any change to routing, DB, or auth must preserve server-derived tenant context.
- **Functional Style**: Leverage `rcs-functional` for iterator chains and state transitions.
- **Migrations**: Add timestamped SQL migrations in `migrations/`. Do not edit `src/schema.rs` manually.
- **Middleware Order**: Tracing -> Auth -> App Handlers. See [src/main.rs](src/main.rs).
- **Keycloak Integration**: Use `KeycloakClient` for OAuth2 flows, handle redirects and callbacks properly.
- **WebSocket**: Use `LogBroadcaster` for real-time logging, configure buffer size appropriately.

## Examples
- **Obtain Tenant Pool**: `let pool = manager.get_pool(&tenant_id)?;`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zlovtnik) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
