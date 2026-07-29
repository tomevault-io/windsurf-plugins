---
trigger: always_on
description: Graph-based workflow orchestrator with unified state model for NullClaw AI bot agents. Part of the Null ecosystem (NullTracker, NullClaw).
---

# NullBoiler

Graph-based workflow orchestrator with unified state model for NullClaw AI bot agents. Part of the Null ecosystem (NullTracker, NullClaw).

## Tech Stack

- **Language**: Zig 0.16.0
- **Database**: SQLite (vendored in `deps/sqlite/`), WAL mode
- **Protocol**: HTTP/1.1 REST API with JSON payloads
- **Dispatch**: HTTP (webhook/api_chat/openai_chat/a2a), MQTT, Redis Streams
- **Vendored C libs**: SQLite (`deps/sqlite/`), hiredis (`deps/hiredis/`), libmosquitto (`deps/mosquitto/`)

## Module Map

| File | Role |
|------|------|
| `main.zig` | CLI args (`--port`, `--db`, `--config`, `--version`, `--export-manifest`, `--from-json`), HTTP accept loop, engine thread, tracker thread |
| `api.zig` | REST API routing and 30+ endpoint handlers (runs, workers, workflows, checkpoints, state, stream snapshots, tracker) |
| `store.zig` | SQLite layer, CRUD methods for all tables, schema migrations (4 migration files) |
| `engine.zig` | Graph-based state scheduler: tick loop, 7 node type handlers, checkpoints, reducers, goto, breakpoints, deferred nodes, reconciliation |
| `state.zig` | Unified state model: 7 reducer types (last_value, append, merge, add, min, max, add_messages), overwrite bypass, ephemeral keys, state path resolution |
| `sse.zig` | Server-Sent Events hub: per-run event queues, 5 stream modes (values, updates, tasks, debug, custom) |
| `dispatch.zig` | Worker selection (tags, capacity, A2A preference), protocol-aware dispatch |
| `async_dispatch.zig` | Thread-safe response queue for async MQTT/Redis dispatch (keyed by correlation_id) |
| `redis_client.zig` | Hiredis wrapper: connect, XADD, listener thread for response streams |
| `mqtt_client.zig` | Libmosquitto wrapper: connect, publish, subscribe, listener thread for response topics |
| `templates.zig` | Prompt template rendering: state-based `{{state.X}}`, legacy `{{input.X}}`, `{{item}}`, `{{task.X}}`, `{{attempt}}`, conditional blocks |
| `callbacks.zig` | Fire-and-forget webhook callbacks on step/run events |
| `config.zig` | JSON config loader (`Config`, `WorkerConfig`, `EngineConfig`, `TrackerConfig`) |
| `types.zig` | `RunStatus`, `StepStatus`, `StepType` (7 types), `WorkerStatus`, `ReducerType`, row types |
| `tracker.zig` | Pull-mode tracker thread: poll NullTickets, claim tasks, heartbeat leases, stall detection |
| `tracker_client.zig` | HTTP client for NullTickets API (claim, heartbeat, transition, fail, artifacts) |
| `workspace.zig` | Workspace lifecycle: create, hook execution, cleanup, path sanitization |
| `subprocess.zig` | NullClaw subprocess: spawn, health check, prompt sending, kill |
| `workflow_loader.zig` | Load JSON workflow definitions from `workflows/` directory, hot-reload watcher |
| `workflow_validation.zig` | Graph-based workflow validation: reachability, cycles, state key refs, route/send targets |
| `ids.zig` | UUID v4 generation, `nowMs()` |
| `metrics.zig` | Prometheus-style metrics counters |
| `strategy.zig` | Pluggable strategy map for workflow execution |
| `worker_protocol.zig` | Protocol-specific request body builders |
| `worker_response.zig` | Protocol-specific response parsers |
| `export_manifest.zig` | Export tool manifest for CLI integration |
| `from_json.zig` | Import workflow from JSON CLI command |

## Build / Test / Run

```sh
zig build              # build
zig build test         # unit tests (355 tests)
zig build && bash tests/test_e2e.sh   # e2e tests (requires Python 3 for mock workers)
./zig-out/bin/nullboiler --port 8080 --db nullboiler.db --config config.json
```

## Step Types (7)

`task`, `route`, `interrupt`, `agent`, `send`, `transform`, `subgraph`

## Reducers (7)

`last_value`, `append`, `merge`, `add`, `min`, `max`, `add_messages`

## API Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/health` | Health check |
| GET | `/metrics` | Prometheus metrics |
| POST | `/workers` | Register worker |
| GET | `/workers` | List workers |
| DELETE | `/workers/{id}` | Remove worker |
| POST | `/runs` | Create workflow run from legacy step-array format |
| GET | `/runs` | List runs (supports ?status= and ?workflow_id= filters) |
| GET | `/runs/{id}` | Get run details |
| POST | `/runs/{id}/cancel` | Cancel run |
| POST | `/runs/{id}/retry` | Retry failed run |
| POST | `/runs/{id}/resume` | Resume interrupted run (with optional state updates) |
| POST | `/runs/{id}/state` | Inject state into running run (pending injection) |
| POST | `/runs/{id}/replay` | Replay run from a checkpoint |
| POST | `/runs/fork` | Fork run from a checkpoint into a new run |
| GET | `/runs/{id}/steps` | List steps for run |
| GET | `/runs/{id}/steps/{step_id}` | Get step details |
| GET | `/runs/{id}/events` | List run events |
| GET | `/runs/{id}/checkpoints` | List checkpoints for run |
| GET | `/runs/{id}/checkpoints/{cpId}` | Get checkpoint details |
| GET | `/runs/{id}/stream` | JSON stream snapshot (supports ?mode=values\|updates\|tasks\|debug\|custom and ?after_seq=) |
| POST | `/workflows` | Create workflow definition |
| GET | `/workflows` | List workflow definitions |
| GET | `/workflows/{id}` | Get workflow definition |
| PUT | `/workflows/{id}` | Update workflow definition |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nullclaw/nullboiler](https://github.com/nullclaw/nullboiler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
