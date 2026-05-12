---
trigger: always_on
description: Portable workflow orchestration for agents and humans.
---

# Stepwise

Portable workflow orchestration for agents and humans.
Package: `stepwise-run` (not `stepwise` — that's taken on PyPI). CLI command: `stepwise`.
Entry point: `stepwise.cli:cli_main` (defined in `pyproject.toml` `[project.scripts]`).

**Push to master = users get it on next `stepwise update`.** There is no staging branch.

**Commit after every meaningful change.** Don't batch unrelated work into one commit. Each commit should be a single logical unit (a feature, a fix, a refactor). Write concise commit messages that explain what changed. This keeps the git log useful for changelog generation and makes reverts safe.

---

## Quick start

```bash
# Python backend
uv sync                                    # install deps
uv run pytest tests/                       # run all tests
uv run pytest tests/test_engine.py         # run one test file
uv run pytest tests/test_engine.py::TestLinearWorkflow::test_linear_a_b_c  # one test
uv run stepwise --help                     # run CLI from dev checkout

# Web frontend (from repo root)
cd web && npm install && npm run dev       # dev server (proxies /api + /ws to localhost:8340)
cd web && npm run test                     # vitest
cd web && npm run lint                     # eslint

# Build & bundle
make build-web                             # npm build → copies web/dist/ → src/stepwise/_web/
```

| CLI mode | What it does |
|---|---|
| `stepwise run <file>` | Headless execution, event-driven engine, exits on job complete/fail. Delegates to running server if one is detected (use `--local` to force standalone). |
| `stepwise run --wait <file>` | Blocking JSON output mode. Delegates to server if available (use `--local` to force standalone). |
| `stepwise run --async <file>` | Fire-and-forget. Delegates to server if available (no subprocess needed); falls back to detached background process with `--local`. |
| `stepwise run --watch <file>` | Launches FastAPI server with auto-created job, opens web UI |
| `stepwise server start` | Persistent web server on port 8340 (REST + WebSocket). `--detach` for background. |
| `stepwise server stop` | Gracefully stop the server |
| `stepwise server restart` | Stop + start (passes through `--detach`, `--port`, etc.) |
| `stepwise server status` | Show PID, port, uptime, log path (or "not running") |
| `stepwise cache stats` | Show cache entries, hits, size, per-flow/step breakdown |
| `stepwise cache clear` | Clear cached results (`--flow`, `--step` filters) |
| `stepwise cache debug <flow> <step>` | Show computed cache key for a step (requires `--input`) |

All delegation modes (`run`, `--wait`, `--async`) use WebSocket notifications from the server for low-latency updates, falling back to REST polling at 2s intervals if WS connection fails.

**Always pass `--name`** when launching jobs. The web UI job list shows the name as the primary label — without it, users see "implement" or "plan-light" for every job with no way to tell them apart. Use a short, human-readable label describing what the job is doing:

```bash
stepwise run flows/implement/FLOW.yaml --name "impl: cost-analytics" --input ...
stepwise run flows/plan-light/FLOW.yaml --name "plan: dag-minimap" --input ...
stepwise job create flows/implement/FLOW.yaml --name "impl: mobile-overhaul" --input ...
```

For plan→implement chains, prefix with `plan:` and `impl:` so the relationship is visible in the list.

---

## Architecture

Python backend (engine, CLI, FastAPI server) + React frontend bundled into the Python package at `src/stepwise/_web/`.

### Module dependency DAG (strict — no circular imports allowed)

```
models → llm_client → executors → engine → server
                                → agent
```

`models.py` must never import from `engine`, `executors`, or `agent`. `executors.py` must never import from `engine`.

### Engine (`src/stepwise/engine.py`)

Two engine classes: `AsyncEngine` (primary, event-driven) and `Engine` (legacy, tick-based).

**AsyncEngine** — event-driven with `asyncio.Queue`. Executors run in the thread pool via `asyncio.to_thread()`. Steps complete → push result event → engine dispatches newly ready steps. Poll watches are driven by `_schedule_poll_watch()` which creates an asyncio task that pushes `poll_check` events at the configured interval.

**Engine** (legacy) — tick-based `engine.tick()` loop. Still used by some tests. All business logic (readiness, exit rules, input resolution) is shared between both engines.

- **Step readiness** (`_is_step_ready()`): no active run + no current completed run (or loop guard) + all deps have current completed runs + `when` condition (if set) evaluates to True
- **Currency** (`_is_current()`): latest run for step is COMPLETED and all dep runs are also current (recursive)
- **Executor dispatch:** `registry.create(ExecutorRef)` → factory lookup → decorator wrapping → `to_thread(executor.start)` (AsyncEngine) or direct call (Engine)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zackham/stepwise](https://github.com/zackham/stepwise) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
