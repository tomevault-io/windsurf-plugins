---
trigger: always_on
description: <!-- Maintenance rule: Only add content that tells AI assistants WHAT TO DO or WHAT NOT TO DO.
---

# AGENTS.md

<!-- Maintenance rule: Only add content that tells AI assistants WHAT TO DO or WHAT NOT TO DO.
     Implementation details, design rationale, and "how the system works" belong in ARCHITECTURE.md.
     If a section doesn't contain an actionable rule or constraint, it doesn't belong here. -->

Project-specific rules and conventions for AI assistants and contributors.

## High-Priority Rules

### Do NOT add fields to `AcpAgentManager` unless every alternative is exhausted

`AcpAgentManager` (in `crates/aionui-ai-agent/src/acp_agent.rs`) is already large and carries multiple overlapping state holders (e.g. `runtime_snapshot`, `state`, `preferred_mode`, `config`). New fields tend to duplicate semantics that `AcpRuntimeSnapshot` or `AcpState` already model, which fragments the source of truth and makes resume/new paths diverge.

Before adding a field:
1. Can the value live in `AcpRuntimeSnapshot`? (runtime/session-scoped state, including user-selected current_mode/current_model/config_selections)
2. Can it be derived from existing fields (`metadata`, `config`, `runtime_snapshot`, `state`)?
3. Can it be persisted via `acp_session.session_config` + `preload_persisted` instead of a new in-memory field?
4. If it must be in-memory and transient, can it be scoped to the call site (local variable, channel, task state) rather than the manager?

Only after exhausting the above — and explicitly documenting why each option is insufficient — add a new field. When doing so, also document its lifecycle (who writes, who reads, when it is invalidated) in a doc comment on the field.

## Architecture

> For detailed background and design decisions, see [ARCHITECTURE.md](./ARCHITECTURE.md).

Cargo workspace organized in four layers: Foundation → Capability → Domain → Composition. Dependencies flow strictly downward.

### Crate Hierarchy & Dependencies

- ✅ Upper layers may depend on lower layers (including cross-layer)
- ✅ Same-layer interaction through trait abstractions only
- ❌ No lower-layer depending on upper-layer
- ❌ No circular dependencies
- Changes to foundation crates require impact assessment

### Domain Crate Structure

Every domain crate must follow:
- `lib.rs` — module exports only, no business logic
- `routes.rs` — export `domain_routes(state) -> Router`, handlers do request/response transformation only
- `service.rs` — sole location for business logic, must not import axum
- `state.rs` — `#[derive(Clone)]` RouterState holding Arc-wrapped dependencies

### API Conventions

- Route prefix: `/api/`
- Resource names: kebab-case
- Response format: `ApiResponse<T>` (success) / `ErrorResponse` (failure)
- All request/response types defined in `aionui-api-types`
- `aionui-api-types` must NOT depend on axum/tower or any HTTP framework

### WebSocket Events

- Format: `domain.camelCaseAction` (two-level structure)
- Message type: `WebSocketMessage<T>` (name + data)
- Existing kebab-case or three-level names are legacy — new events must follow the convention

### Data Layer

- Repository traits in `aionui-db`, prefixed with `I`
- Concrete implementations prefixed with `Sqlite`
- Row models in `aionui-db/src/models/`
- Params objects co-located in repository files
- Migrations: `NNN_descriptive_name.sql`, no manual DB modifications
- Services depend on traits, never on concrete implementations

### Dependency Injection

- `AppServices` is the sole service construction center
- Domain crates only define RouterState, never construct their own dependencies
- All assembly happens in `aionui-app`'s `build_*_state()` functions

### Security

- New endpoints must be evaluated for auth middleware requirement
- State-changing operations must be CSRF-protected
- Sensitive operations should have rate limiting
- Error responses must not leak internal details
- Secrets must never be hardcoded

## Code Style

- Rust 2024 edition, stable toolchain (pinned in `rust-toolchain.toml`)
- Comments in English, commit messages in English
- Each `.rs` file follows single responsibility — one module, one concern
- Max 1000 lines per `.rs` file; split into submodules when approaching the limit

## Development Workflow

### Subprocess Spawning

New subprocess spawn sites must use `aionui_runtime::Builder::agent(program)` or `aionui_runtime::Builder::clean_cli(program)`. Do NOT use raw `tokio::process::Command`. See [ARCHITECTURE.md § Runtime Infrastructure](./ARCHITECTURE.md#runtime-infrastructure) for details.

### Pushing Code

Always use `just push` instead of `git push`.
It runs fmt → clippy → test before pushing, preventing CI failures.
Supports the same arguments as `git push` (e.g. `just push -u origin feat/branch`).

### Add Endpoint to Existing Crate

1. Request/response types → `aionui-api-types/src/{domain}.rs`
2. Handler function → `crates/aionui-{domain}/src/routes.rs`
3. Business logic → `crates/aionui-{domain}/src/service.rs`
4. Register route in `domain_routes()` function
5. Add test → `crates/aionui-{domain}/tests/` or `crates/aionui-app/tests/`

### Add Migration

1. Next number → `ls crates/aionui-db/migrations/`
2. Create `NNN_descriptive_name.sql` with `IF NOT EXISTS`

### Add WebSocket Event

1. Event type → `aionui-api-types`
2. Emit via `event_bus.broadcast()` in service

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iOfficeAI/AionCLI](https://github.com/iOfficeAI/AionCLI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
