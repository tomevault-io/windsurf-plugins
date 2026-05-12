---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development Commands

```bash
# Build all workspace members
cargo build --release

# Check code (faster than build)
cargo check

# Run tests
cargo nextest run

# Run clippy lints
cargo clippy --all-targets

# Run the engine with example config
./target/release/emergent --config ./config/emergent.toml

# Run a single example primitive (for testing)
cargo run --release -p timer -- --interval 5000
cargo run --release -p filter -- --filter-every 5
cargo run --release -p console
cargo run --release -p log -- --output ./timer_events.log
cargo run --release -p exec

# Scaffold a new primitive (interactive or scripted)
emergent scaffold
emergent scaffold -t handler -n my_filter -l rust -S timer.tick -p timer.filtered

# Initialize a new config file
emergent init

# Marketplace commands
emergent marketplace list
emergent marketplace install http-source
```

## Architecture Overview

Emergent is an **event-driven workflow engine** built on **acton-reactive** (a Rust actor framework). It implements a publish-subscribe pattern using three primitive types that communicate via Unix IPC sockets.

### Core Components

```
┌──────────────────────────────────────────────────────────────────────┐
│                        Emergent Engine                               │
│  ┌─────────────┐  ┌─────────────┐  ┌───────────────┐  ┌──────────┐  │
│  │  Process    │  │    IPC      │  │  Event Store  │  │ HTTP API │  │
│  │  Manager    │  │   Server    │  │ (JSON+SQLite) │  │ (Axum)   │  │
│  └─────────────┘  └─────────────┘  └───────────────┘  └──────────┘  │
└──────────────────────────────────────────────────────────────────────┘
        │                 │                                   │
        ▼                 ▼                                   ▼
   ┌─────────┐      ┌───────────┐      ┌────────┐     /api/topology
   │ Sources │      │ Handlers  │      │ Sinks  │
   └─────────┘      └───────────┘      └────────┘
```

### Startup & Shutdown Order

- **Startup**: Sinks → Handlers → Sources (consumers ready before producers)
- **Shutdown**: Sources (SIGTERM) → Handlers (`system.shutdown` broadcast) → Sinks (`system.shutdown` broadcast)

### The Three Primitives

| Primitive | Capabilities | Purpose |
|-----------|-------------|---------|
| **Source** | Publish only | Ingress - emit events into the system |
| **Handler** | Subscribe + Publish | Transform - process and re-emit events |
| **Sink** | Subscribe only | Egress - consume events (logs, console, HTTP) |

### Workspace Structure

- **emergent-engine** (`emergent-engine/`): Core runtime, process manager, message broker, event store, scaffold, marketplace
- **emergent-client** (`sdks/rust/`): Rust SDK for building Sources, Handlers, and Sinks
- **sdks/ts**: TypeScript/Deno SDK
- **sdks/py**: Python SDK (uses `uv` for package management)
- **sdks/go**: Go SDK
- **examples/sources/**: timer (Rust), timer-go (Go), webhook (Python), topology-api (TypeScript)
- **examples/handlers/**: filter (Rust), filter-go (Go), exec (Rust), topology-api (TypeScript)
- **examples/sinks/**: console (Rust), console-go (Go), log (Rust), console_color (TypeScript), webhook_console (Python), topology-viewer (TypeScript)

### Engine Modules

- `config.rs` — TOML config loading, path expansion, validation
- `process_manager.rs` — Actor-based lifecycle for primitives
- `primitive_actor.rs` — Per-primitive actor (spawns child process, monitors, broadcasts system events)
- `event_store/` — JSON append-only logs + SQLite structured storage
- `scaffold/` — Code generation for new primitives (Rust, Python, TypeScript templates)
- `marketplace/` — Registry client for discovering and installing community primitives
- `init/` — Interactive `emergent init` to create emergent.toml

### Key Abstractions

**EmergentMessage** (`sdks/rust/src/message.rs`) - The universal message envelope:
```rust
pub struct EmergentMessage {
    pub id: MessageId,                    // TypeID: msg_<UUIDv7>
    pub message_type: MessageType,        // e.g., "timer.tick"
    pub source: PrimitiveName,            // primitive name
    pub correlation_id: Option<CorrelationId>,
    pub causation_id: Option<CausationId>,  // enables event tracing
    pub timestamp_ms: Timestamp,          // Unix ms
    pub payload: serde_json::Value,
    pub metadata: Option<serde_json::Value>,
}
```
Types are in `sdks/rust/src/types/` — `MessageId`, `MessageType`, `PrimitiveName`, `CorrelationId`, `CausationId`, `Timestamp`.

**System Events** — Engine broadcasts lifecycle events:
- `system.started.<name>` - primitive started successfully
- `system.stopped.<name>` - primitive stopped gracefully
- `system.error.<name>` - primitive failed
- `system.shutdown` - signals primitives to gracefully stop
- `system.request.subscriptions` / `system.response.subscriptions` - SDK subscription discovery
- `system.request.topology` / `system.response.topology` - topology queries via pub/sub

### IPC Protocol

- Wire format: MessagePack (default) or JSON
- Transport: Unix domain sockets
- Messages registered with `#[acton_message(ipc)]` macro from acton-reactive

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Govcraft/emergent](https://github.com/Govcraft/emergent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
