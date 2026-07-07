---
trigger: always_on
description: > Universal instruction file for AI coding assistants (Claude, Codex, Cursor, Copilot, Windsurf, etc.)
---

# Foundation Intelligence

> Universal instruction file for AI coding assistants (Claude, Codex, Cursor, Copilot, Windsurf, etc.)

**New here?** Start with [`README.md`](README.md) → [`docs/PHILOSOPHY.md`](docs/PHILOSOPHY.md) → [`docs/foundation_quick_start.md`](docs/foundation_quick_start.md), then come back to this file for the agent operating contract.

## Terminology

| Term | Definition |
| :--- | :--- |
| **Foundation Core** | The shared infrastructure repository containing `server-kit`, `runtime-transport`, `runtime-sdk`, etc. |
| **Foundation Template** | The skeletal structure in `templates/` used to bootstrap new projects. |
| **Foundation Project** | A specific application (e.g., `trader_os`, `fintech_v1`) generated from the template. |
| **Foundation Reference** | The `/foundation` directory inside a **Project**, which is a local copy/reference to Core modules. |

## Project Context

This is a **Foundation** project - a production-grade full-stack application using Go backend, TypeScript/React frontend, and Rust/WASM for high-performance compute. The foundation provides shared infrastructure for event-driven, tenant-isolated, realtime applications.

## Agent Operating Baseline

Before editing architecture-sensitive code, read these files in order:

1. `docs/foundation_glossary.md` or `docs/foundation/foundation_glossary.md` — concept lookup and agent Q&A
2. `docs/foundation_quick_start.md` or `docs/foundation/foundation_quick_start.md`
3. `docs/foundation_tour.md` or `docs/foundation/foundation_tour.md`
4. `docs/foundation_architecture_contract.md` or `docs/foundation/foundation_architecture_contract.md`
5. `docs/agent_operating_contract.md` or `docs/foundation/agent_operating_contract.md`
6. `docs/practice_controls.md` or `docs/foundation/practice_controls.md`
7. `docs/ai_threat_model.md` or `docs/foundation/ai_threat_model.md` when tool, model, retrieved, generated, package, or security-sensitive input affects the change
8. The relevant practice file for the lane you are changing
9. `docs/future_practices_research.md` or `docs/foundation/future_practices_research.md` when proposing a new practice, security posture, performance lane, or agent workflow

Definition of Done for agent-authored changes:

1. State whether a public contract changed.
2. Identify the invariant that must still hold.
3. Leave evidence: test, benchmark, static check, review note, or migration proof.
4. Preserve or document the fallback path.
5. Name the scope boundary touched.
6. Add or update a regression guard.
7. Update docs or explain why no documentation changed.

## Tech Stack (2026 Standards)

| Layer | Technology | Version |
| ------- | ------------ | --------- |
| Backend | Go | 1.26+ |
| Frontend | TypeScript, React | 5.9+, 19.2+ |
| High-Performance | Rust, WASM | 1.95+ |
| Database | PostgreSQL | 18+ |
| Cache/Pubsub | Redis | 8+ |
| Queue | River (Go) | Latest |
| Protocol | Protocol Buffers, Cap'n Proto | 3.x |

## Architecture Overview

A **Foundation Project** is structured to separate shared infrastructure from application logic:

```text
project/
├── cmd/                    # Entry points (server, worker)
├── internal/               # Application logic
│   ├── service/           # Domain services (e.g., service/order, service/user)
│   ├── worker/            # Background job handlers
│   └── startup/           # App-specific initialization
├── api/                    # Protocol definitions
│   ├── protos/            # App-specific Protobuf schemas
│   └── schemas/           # Cap'n Proto schemas
├── frontend/              # React SPA
│   └── src/
│       ├── features/      # Feature modules
│       ├── components/    # Shared UI (wrapping foundation primitives)
│       ├── runtime/       # WASM bridge, transport adapters
│       └── stores/        # State management (Zustand + transport)
├── docs/
│   └── foundation/        # Foundation practices (copied from Core)
├── foundation/            # Shared infrastructure (READ-ONLY REFERENCE)
│   ├── server-kit/go/     # Backend modules
│   ├── runtime-transport/ # Client transport
│   ├── runtime-sdk/       # WASM kernel
│   ├── ui-minimal/        # UI Primitives
│   └── frontend-kit/      # Frontend operational utilities
└── rust/                  # Native Rust crates
```

## Critical Rules (Mandatory)

### 1. Foundation Dependency Boundary

**NEVER** import or alias raw source files from `foundation/*/ts/src` or internal Go packages.

- **Frontend**: Consume foundation logic via package boundaries: `@ovasabi/runtime-transport`, `@ovasabi/frontend-kit`, `@ovasabi/ui-minimal`.
- **Backend**: Use the `server-kit` module exports.

### 2. Correlation ID Propagation

Every mutating command MUST carry a `correlationId`. Trace it through all workers, events, and logs.

```go
// CORRECT
ctx = metadata.WithCorrelationID(ctx, envelope.CorrelationID)
bus.Publish(ctx, "domain:action:requested", payload)
```

### 3. Event Contract Lifecycle

All domain events follow `<domain>:<action>:<state>` pattern:

- `:requested` - Command received, validation passed
- `:success` - Operation completed
- `:failed` - Operation failed with reason

### 4. Tenant Isolation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nmxmxh/foundation](https://github.com/nmxmxh/foundation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
