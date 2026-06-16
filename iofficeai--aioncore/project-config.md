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

### Logging

When planning or changing a critical path or hard-to-observe flow, evaluate whether logging needs to change. In implementation plans for such changes, briefly state whether logs will be added, existing observability is sufficient, or logs are intentionally unnecessary. Do not add logs for simple refactors, test-only changes, UI copy/style changes, or when existing tests, errors, metrics, or logs already provide enough observability.

Add structured logs only when they help make production behavior diagnosable or provide extra development detail. Production normally runs at `info`, while development runs at `debug`; therefore, information needed to troubleshoot production issues must be available at `info`, `warn`, or `error`, not only `debug`.

Use log levels as follows:
- `debug` for high-frequency or detailed development-only flow details and state transitions
- `info` for low-volume production-diagnostic lifecycle boundaries, important state changes, and non-sensitive correlation context
- `warn` for malformed or unexpected data that is safely handled
- `error` for contract violations or failed operations

Production-visible logs must not include sensitive payloads such as prompts, tool input/output, file contents, command bodies, tokens, secrets, or raw provider requests/responses. If such payloads are needed for local debugging, they must be behind explicit development-only guards and never enabled by default.

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
- Use `aionui_common::ApiError` only at API/HTTP boundaries such as routes and middleware. Service/domain code must prefer crate-owned errors (`ConversationError`, `TeamError`, etc.) and map them to `ApiError` in route modules. Do not introduce new `AppError` usages; it exists only as a temporary compatibility alias.

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

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iOfficeAI/AionCore](https://github.com/iOfficeAI/AionCore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
