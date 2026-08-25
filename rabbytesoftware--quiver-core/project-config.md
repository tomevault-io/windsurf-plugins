---
trigger: always_on
description: > **How to use this file:** It describes concepts, patterns, and conventions — not current implementations. When you need exact method signatures or struct fields, read the actual source file. This file tells you WHERE to look and HOW things are wired, so you can derive the correct implementation from current code.
---

# quiver.core — Codebase Knowledge

> **How to use this file:** It describes concepts, patterns, and conventions — not current implementations. When you need exact method signatures or struct fields, read the actual source file. This file tells you WHERE to look and HOW things are wired, so you can derive the correct implementation from current code.

---

## 1. What Quiver Is

Quiver is a decentralised, event-sourced package manager. It installs and manages software through Git repositories. The `quiver.core` daemon exposes REST and WebSocket APIs for managing:

- **Arrows** — packages (manifests fetched from Git repos)
- **Collections** — curated catalogs of arrows
- **Runtimes** — execution contexts (install, run, stop, uninstall)

Module path: `github.com/rabbytesoftware/quiver.core`

---

## 2. Layer Architecture

Six layers under `internal/`, one binary at `cmd/quiver/`. **Dependencies flow strictly downward** — upper layers never import lower layers upward.

```
cmd/quiver/          ← Cobra CLI entry point
internal/internal.go ← DI wiring (New + Start)
internal/api/        ← HTTP + WebSocket delivery (Gin)
internal/app/        ← Orchestration: usecases, repositories, hub
internal/engine/     ← Stateless business engines
internal/adapter/    ← Storage backends (SQLite via Asynx + GORM)
internal/core/       ← Process singletons: config, paths, logger, fns
internal/domain/     ← Pure types and state machines (no I/O, no internal imports)
```

### Layer responsibilities

| Layer | Key rule |
|-------|----------|
| `domain/` | No I/O. No imports from other internal packages. Pure types + state machines. |
| `core/` | Config, embedded metadata, path resolution, logger, FetchNShare I/O. |
| `adapter/` | Asynx event store (SQLite) + generic `Store[T,K]` (sqlite/memory). |
| `engine/` | Manifold, Vault, Wizard, DepTree, Netbridge. Each is independent — no engine imports another. |
| `app/` | Owns Asynx aggregates, composes engines + adapters into usecases, owns `WebSocketHub`. |
| `api/` | Gin routes, Gorilla WebSocket. Maps HTTP ↔ usecase calls ↔ DTOs. Knows nothing about Asynx/commands/projections. |

### DI construction order (in `internal.New`)

```
engine.New(ctx)
adapter.New()
app.New(engines, adapters)
apiv0.New(appContainer)
api.New(appContainer.Hub, buildInfo, v0Container)
```

Read `internal/internal.go` for the current wiring.

---

## 3. Domain Model

Read files under `internal/domain/` for current struct fields — they change without notice. Below are stable descriptions of what each type represents.

### 3.1 Namespace

A string type (`domain.Namespace`) in `domain/user/repo` format, optionally with `@ref` and optionally with a 4th segment for quiver-hosted arrows (`domain/user/repo/auid`). Key operations: strip ref, extract ref, replace ref, validate, extract segments (QUID = first 3, AUID = 4th), derive clone URL.

Read `internal/domain/namespace.go` for exact methods.

### 3.2 Arrow

The canonical installed package aggregate. Holds the compiled manifest (name, description, version, variables, netbridge port definitions, per-OS targets) plus installation metadata (installed ref, constraint, timestamp).

**ArrowState** is a string enum with the following valid transitions:

| From | Can transition to |
|------|------------------|
| `absent` | `ready` |
| `ready` | `running`, `installing`, `uninstalling`, `updating`, `outdated` |
| `running` | `stopping`, `detached` |
| `stopping` | `ready`, `draining` |
| `draining` | `ready` |
| `detached` | `ready`, `stopping` |
| `installing` | `ready`, `absent` |
| `uninstalling` | `absent`, `ready` |
| `updating` | `ready`, `absent` |
| `outdated` | `ready`, `uninstalling` |
| `removed` | (terminal) |

`IsActive()` returns true for: `running`, `stopping`, `draining`, `installing`, `updating`.

### 3.3 ArrowRuntime

Execution state for an arrow. Tracks current state, active execution (method being run, progress steps, PID, workdir, variables), and last return value (outcome, steps, variables). Lives in `internal/domain/runtime/`.

### 3.4 Collection

A followed catalog of arrows. Holds namespace, follow timestamp, list of arrows in the collection, and collection metadata.

### 3.5 Target

Per-OS entry inside an Arrow manifest. Contains hardware requirements, tool/service dependency edges, exported environment values, and a lifecycle map (`install`, `update`, `execute`, `stop`, `uninstall`, user-defined methods).

### 3.6 Lifecycle method constants

```
_install   _uninstall   _update   _execute   _stop
```

These are the names passed to `wizard.Start` and appear as method keys in Arrow.Targets.

### 3.7 OS identifiers

String enum covering: `linux/amd64`, `linux/arm64`, `windows/amd64`, `windows/arm64`, `darwin/amd64`, `darwin/arm64`. `domain.CurrentOS()` returns the running platform.

---

## 4. Event Sourcing — Asynx Pattern

The project uses `github.com/char2cs/asynx`. Every state change flows through a Command.

### 4.1 Command contract


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rabbytesoftware/quiver.core](https://github.com/rabbytesoftware/quiver.core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
