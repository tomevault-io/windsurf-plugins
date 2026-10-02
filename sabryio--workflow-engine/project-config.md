---
trigger: always_on
description: A durable workflow engine with a Rust core and network API. Combines event-driven fan-out with direct invocation as one unified surface.
---

# Workflow Engine

A durable workflow engine with a Rust core and network API. Combines event-driven fan-out with direct invocation as one unified surface.

## Architecture

Clean Architecture with explicit layer boundaries:

```
workflow-engine/
├── src/                          # Core library (Clean Architecture layers)
│   ├── domain/                   # Pure business logic (no dependencies on other layers)
│   │   ├── workflow.rs           # Workflow trait, ErasedWorkflow
│   │   ├── context.rs            # Context API
│   │   ├── event.rs              # Event trait
│   │   ├── realtime.rs           # RealtimeContext (domain API wrapper)
│   │   └── types/                # Domain value objects
│   │       ├── ids.rs            # RunId
│   │       ├── run.rs            # RunSnapshot, RunStatus, StepRecord
│   │       ├── error.rs          # WorkflowError
│   │       └── retry.rs          # RetryPolicy, WorkflowConfig
│   ├── ports/                    # Trait abstractions (dependency inversion)
│   │   └── traits.rs             # StateStore, Clock, EventPublisher, EventSubscriber,
│   │                             # RealtimeBus, WorkflowInvoker
│   ├── adapters/                 # Infrastructure implementations
│   │   └── memory/               # Built-in memory adapters
│   │       ├── state.rs          # MemoryStateStore
│   │       ├── event_bus.rs      # MemoryEventBus
│   │       ├── clock.rs          # RealClock, TestClock
│   │       └── realtime.rs       # BroadcastRealtimeBus
│   └── application/              # Use case orchestration
│       ├── orchestrator.rs       # Orchestrator + test utilities
│       ├── registry.rs           # Workflow registration
│       ├── admin.rs              # AdminClient
│       ├── remote.rs             # RemoteHub
│       └── remote_workflow.rs    # RemoteWorkflow
└── examples/
    └── http-server/              # HTTP API example (optional) — Clean Architecture
        ├── src/
        │   ├── bootstrap/        # Configuration & logging initialization
        │   │   ├── config.rs     # Config struct (host, port, log_level)
        │   │   ├── env.rs        # Environment defaults
        │   │   └── logging.rs    # Tracing subscriber setup
        │   ├── composition/      # Dependency wiring
        │   │   ├── app_state.rs  # AppState + FromRef impls
        │   │   └── builder.rs    # Dependency graph construction
        │   ├── dto/              # Data Transfer Objects (HTTP contracts)
        │   │   ├── error.rs      # ApiError, ApiWrap, error responses
        │   │   ├── workflow.rs   # Trigger/Schedule/Publish DTOs
        │   │   ├── run.rs        # Run management DTOs
        │   │   ├── worker.rs     # Worker registration DTOs
        │   │   ├── admin.rs      # Admin operation DTOs
        │   │   └── realtime.rs   # Realtime/SSE DTOs
        │   ├── handlers/         # HTTP request handlers (transport layer)
        │   │   ├── workflow.rs   # trigger, schedule, publish
        │   │   ├── run.rs        # get_run, list_runs, history, write_step, cancel, retry
        │   │   ├── worker.rs     # register_worker, unregister_worker, lease, complete
        │   │   ├── admin.rs      # pause, resume, drain, dead_letters
        │   │   ├── health.rs     # health, metrics
        │   │   ├── event.rs      # wait_event (long-poll)
        │   │   └── realtime.rs   # subscribe_channel, realtime_publish
        │   ├── server.rs         # Router assembly (pure function: AppState → Router)
        │   └── main.rs           # Entry point (bootstrap → composition → server → TCP)
```

**Key Point**: The core `workflow-engine` library has NO HTTP dependencies. HTTP is just one example of how to expose the engine. You can embed the orchestrator directly in your application, or build adapters for CLI, gRPC, message queues, Lambda, etc.

**Dependency flow**: domain ← ports ← adapters/application. Domain has zero dependencies on infrastructure or orchestration.

### Project Structure

Single library crate with Clean Architecture layers:

- **src/domain/** - Pure business logic: Workflow trait, Context API, domain types (RunId, WorkflowError, etc.)
- **src/ports/** - Trait abstractions for dependency inversion (StateStore, Clock, EventPublisher, etc.)
- **src/adapters/** - Infrastructure implementations (memory/, future: postgres/)
- **src/application/** - Use case orchestration (Orchestrator, Registry, AdminClient, RemoteHub)
- **examples/http-server/** - HTTP server exposing the engine using Clean Architecture:
  - **bootstrap/** - Configuration loading, logging setup, environment parsing
  - **composition/** - Dependency wiring (orchestrator, registry, hub, buses)
  - **dto/** - HTTP request/response types (API contracts)
  - **handlers/** - HTTP request handlers grouped by feature area
  - **server.rs** - Pure router assembly function
  - **main.rs** - Entry point orchestrating bootstrap → composition → server → TCP
- **tests/** - Inline in `src/application/orchestrator.rs` under `#[cfg(test)] pub mod test`

**TypeScript SDK** at `sdks/typescript/` - Client + worker runtime, package name `@workflow-engine/client`

## Setup

### Prerequisites

- Rust 1.75+ (edition 2021)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sabryio/workflow-engine](https://github.com/sabryio/workflow-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
