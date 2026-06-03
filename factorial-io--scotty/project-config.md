---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Scotty is a micro Platform-as-a-Service (PaaS) for managing Docker Compose-based applications:

- **scotty**: HTTP server (REST API + WebSocket) for managing Docker Compose apps
- **scottyctl**: CLI client for the scotty server
- **scotty-core**: Shared business logic (Docker operations, settings, tasks)
- **scotty-types**: Shared type definitions (TypeScript-compatible via ts-rs)
- **frontend**: SvelteKit web interface (tightly coupled with API, no backwards compatibility needed)
- **ts-generator**: Utility for generating TypeScript bindings from Rust types

## Development Commands

```bash
# Tests
cargo test                                              # Run all tests
cargo test test_name -- --nocapture                     # Specific test with output
RUST_LOG=debug cargo test test_name -p scotty -- --nocapture  # With debug logging

# Server (use .env file for SCOTTY__API__AUTH_MODE=dev etc.)
SCOTTY__API__AUTH_MODE=dev cargo run --bin scotty        # Dev mode (no auth)
RUST_LOG=info cargo run --bin scotty                     # With logging
cargo run --bin scotty -- config                         # View configuration

# scottyctl
cargo run --bin scottyctl -- <command>
cargo run --bin scottyctl -- --server http://localhost:21342 --access-token <token> app:list
# Or via env: SCOTTY_SERVER=http://localhost:21342 SCOTTY_ACCESS_TOKEN=<token>

# Frontend (uses bun, not npm)
cd frontend && bun install && bun run dev               # Development server
bun run build                                           # Production build
bun run check                                           # Type checking
bun run lint                                            # Prettier + ESLint (must pass before push)

# Prerequisites: start Traefik for local development
cd apps/traefik && docker compose up -d
```

## Architecture

### Scotty Server (`scotty/src/`)

**Entry Point**: `main.rs` — initializes AppState (settings, Docker client, task manager), sets up OpenTelemetry, spawns HTTP server and background tasks.

**Key Modules**:
- `api/router.rs`: Axum router with OpenAPI docs (utoipa)
- `api/rest/handlers/`: REST endpoints — `apps/` (create, list, run, actions, notifications), `admin/` (assignments, permissions, roles, scopes), `scopes/` (user-facing), `blueprints.rs`, `landing.rs` (Traefik fallback routing), `login.rs`, `tasks.rs`, `health.rs`, `info.rs`
- `api/websocket/`: Real-time features — `handlers/` (auth, logs, shell, tasks), `messaging.rs` (protocol), `client.rs` (connection mgmt)
- `api/auth_core.rs`: Core authentication logic
- `api/middleware/`: Casbin RBAC authorization
- `api/rate_limiting/`: Per-tier rate limiting
- `docker/state_machine_handlers/`: App lifecycle steps (create dir, save files, docker login, compose up, load balancer config, post actions, wait for containers, etc.)
- `docker/services/`: Long-running log streaming and shell sessions
- `docker/loadbalancer/`: Traefik/HAProxy config generation
- `onepassword/`: 1Password secrets — resolves `op://` URIs in app env vars (two-pass: 1Password lookup, then env var substitution)
- `oauth/`: OAuth 2.0 — device flow (CLI) and web flow (`/oauth/authorize`, `/api/oauth/callback`, `/oauth/exchange`)
- `services/authorization/`: Casbin RBAC (scopes, roles, permissions)
- `tasks/`: Task execution and output streaming
- `notification/`: Log, Webhook, Mattermost, GitLab notifications
- `static_files.rs`: Embedded frontend serving
- `metrics/`: Collectors for app list, HTTP requests, memory usage, Tokio runtime, etc.

**AppState** (shared via `Arc`): Settings, Docker client (Bollard), task manager, authorization service, metrics collectors.

### Authorization System

Uses Casbin for RBAC. Config: `config/casbin/policy.yaml`. Implementation: `scotty/src/services/authorization/casbin.rs`. Tests: `scotty/tests/authorization_domain_test.rs`.

**Permissions**: `view`, `manage`, `create`, `destroy`, `shell`, `logs`, `admin_read`, `admin_write`, `action_read`, `action_write`, `action_manage`, `action_approve`

**Assignment matching** (by precedence): exact email (`user@factorial.io`) > domain pattern (`@factorial.io`) > wildcard (`*`). Wildcard is always additive. Domain patterns prevent subdomain attacks. Case-insensitive per RFC 5321.

```yaml
# config/casbin/policy.yaml
scopes:
  client-a: { description: "Client A Production" }
  qa: { description: "QA Environment" }
roles:
  admin: { permissions: ['*'], description: "Full access" }
  developer: { permissions: ['view', 'manage', 'create', 'shell', 'logs'], description: "Dev access" }
  viewer: { permissions: ['view'], description: "Read-only" }
assignments:
  stephan@factorial.io:                     # Exact match (highest priority)
    - { role: admin, scopes: ['*'] }
  '@factorial.io':                          # Domain match (fallback)
    - { role: developer, scopes: ['client-a', 'qa'] }
  '*':                                      # Wildcard (always additive)
    - { role: viewer, scopes: ['default'] }
```

### scottyctl (`scottyctl/src/`)

**Commands** (colon-separated namespace):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [factorial-io/scotty](https://github.com/factorial-io/scotty) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
