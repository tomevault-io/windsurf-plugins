---
trigger: always_on
description: This project uses OpenWolf for context management. Read and follow .wolf/OPENWOLF.md every session. Check .wolf/cerebrum.md before generating code. Check .wolf/anatomy.md before reading files.
---

# OpenWolf

@.wolf/OPENWOLF.md

This project uses OpenWolf for context management. Read and follow .wolf/OPENWOLF.md every session. Check .wolf/cerebrum.md before generating code. Check .wolf/anatomy.md before reading files.


# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Module

`github.com/Vilsol/klados` — Go 1.25, Wails v3 alpha.74

## Monorepo

pnpm workspace (`pnpm-workspace.yaml`): `frontend/`, `packages/*`, `apps/*`. Use `pnpm install` (not npm).

Tool versions managed by `mise.toml` (wails3, go-jsonschema, node 25, tinygo, pnpm).

## Build

```bash
# Dev mode (hot reload, starts Vite + Go watcher)
task dev

# Frontend only
cd frontend && pnpm install && pnpm build

# Go binary (requires CGO for Wails/GTK on Linux)
go build .

# Regenerate Wails bindings (after any Go service signature change):
wails3 generate bindings

# Regenerate plugin types from JSON Schemas:
mise run generate:plugin-types

# Type-check frontend:
cd frontend && pnpm check
```

## Test

```bash
# Go — packages that don't need CGO (fast)
go test ./internal/config/ ./internal/session/ ./internal/cluster/ ./internal/streaming/ ./internal/watcher/ -v

# Go — all packages (CGO required, imports Wails)
go test ./internal/... -v

# Single Go test
go test ./internal/cluster/ -run TestLoadKubeconfigs -v

# Integration tests (requires a live cluster)
go test ./internal/... -v -tags integration

# Frontend
cd frontend && pnpm test

# Single frontend test file
cd frontend && npx vitest run src/lib/__tests__/Header.svelte.test.ts
```

## Architecture

### Data flow

```
Kubernetes API → cluster.Manager (dynamic client)
                      ↓
              resource.ResourceEngine  →  List/Get/Delete via dynamic.Interface
              resource.EnricherRegistry →  per-GVR Enricher injects computed fields
                      ↓
              watcher.WatchManager    →  emits watch:{ctx}:{gvr}:{ns} Wails events
                      ↓
              services.ResourceService →  Wails-bound RPC layer
                      ↓
              frontend ResourceStore  →  subscribes to events, owns items[]
                      ↓
              ResourceList.svelte     →  TanStack Virtual, CEL column rendering
```

### Go backend (`internal/`)

| Package | Responsibility |
|---|---|
| `logging/` | `Setup()` returns `context.Context` with tint-backed slog logger via slox. Use `slox.Info(ctx, ...)` everywhere — never pass `*slog.Logger` directly. |
| `config/` | JSON config at `$XDG_CONFIG_HOME/klados/config.json` |
| `session/` | State at `$XDG_STATE_HOME/klados/session.json`, debounced 500ms save |
| `cluster/` | `Manager`: kubeconfig loading, connect/disconnect, health monitor (15s), `DiscoverResources()` emits `discovery:{ctx}:resources` on connect |
| `resource/` | `Registry` (CEL-validated descriptors), `ResourceEngine` (List/Get/Delete), `EnricherRegistry` + per-resource enrichers that inject display fields into unstructured objects |
| `watcher/` | `WatchManager`: start/stop per `(ctx, gvr, namespace)` key; 30s grace period before actually stopping; emits `watch:{ctx}:{gvr}:{ns}` events with `{type, object}` payload |
| `streaming/` | Fiber HTTP server on random localhost port, token auth, emits `streaming:ready` with port+token |
| `logs/` | `LogStreamer`: per-container log streaming over WebSocket, 1024-item buffered channel for backpressure |
| `exec/` | `ExecManager`: interactive shell sessions via WebSocket, resize via text JSON frames |
| `portforward/` | `Manager`: port-forward lifecycle, emits `portforward:{ctx}:{id}` (per-forward) and `portforward:{ctx}:updated` (aggregate) events |
| `metrics/` | Metrics collection and aggregation |
| `services/` | Wails service layer — `AppService` owns `cluster.Manager` and `streaming.Server`; `ResourceService` owns `ResourceEngine` and `WatchManager` |
| `plugin/` | Plugin system: wazero Wasm runtime, manifest validation, permission enforcement, enricher adapter, hot reload via fsnotify. See [PLUGIN_ARCHITECTURE.md](PLUGIN_ARCHITECTURE.md) for full spec. |

### GVR format

Dot-separated: `apps.v1.deployments`, `core.v1.pods`, `networking.k8s.io.v1.ingresses`. The `core` prefix replaces empty group. `ParseGVR()` splits from the right (handles groups with dots).

### Three-stage rendering pipeline

1. **Go enricher** — injects computed fields into `unstructured.Unstructured` (e.g. `status.readyDisplay`, `status.restartCount`)
2. **CEL extraction** — column `expr` strings evaluated at render time via `cel-js` `evalExpr(expr, obj)`
3. **Frontend renderer** — `renderType`: `text` | `badge` | `age` | `progress`

Adding a new resource type requires: a `Descriptor` in `internal/resource/builtin.go` (optional enricher), and the descriptor is automatically serialized to the frontend via `GetDescriptors()`. Unknown GVRs get a fallback descriptor (Name, Namespace, Age).

### Wails events

All callbacks receive `WailsEvent { name, data }` — always unwrap with `wailsEvent.data`. `Events.On()` returns an unsubscribe function; use it instead of `Events.Off()`.

### Frontend (`frontend/src/`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Vilsol/klados](https://github.com/Vilsol/klados) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
