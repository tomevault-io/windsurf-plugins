---
trigger: always_on
description: Envoi is a monorepo containing an SDK for building API-backed evaluation environments, a coding agent evaluation framework, and a unified CLI. It uses **uv workspaces** for Python package management.
---

# AGENTS.md

## Repository Overview

Envoi is a monorepo containing an SDK for building API-backed evaluation environments, a coding agent evaluation framework, and a unified CLI. It uses **uv workspaces** for Python package management.

### Packages

| Package | Install name | What it does |
|---------|-------------|--------------|
| `packages/envoi/` | `envoi` | SDK for authoring evaluation environments (`@envoi.suite`, `@envoi.test`, `@envoi.setup`) |
| `packages/code/` | `envoi-code` | Orchestrates coding agents against envoi environments, captures parquet traces |
| `packages/cli/` | `envoi-cli` | Unified `envoi` CLI. Routes subcommands to the right packages |

Dependency graph:

```
envoi-cli  ──→  envoi-code  ──→  envoi
                                    ↑
envoi-cli  ─────────────────────────┘
```

### Examples

Examples live in `examples/<name>/` with colocated `task/` and `environment/` directories:

```
examples/
├── c_compiler/
│   ├── task/
│   │   ├── task.py
│   │   └── prompt.md
│   └── environment/
│       ├── main.py
│       ├── Dockerfile
│       ├── params.py
│       └── tests/
└── polish_notation/
    └── environment/
```

## Vocabulary (Canonical)

- `part`:
  - Most granular observable unit in the trace.
  - A meaningful assistant action: `reasoning`, `text`, `tool`, `tool_use`, `tool_result`, or `patch`.
  - Global part index is the authoritative progress counter.
  - Budgeting and limits are based on parts (`--max-parts`).
- `turn`:
  - One request/response loop in the orchestrator.
  - A turn can contain many parts, one part, or zero meaningful parts.
  - Turns are grouping metadata only, not budgeting/accounting units.
- `step`:
  - Forbidden term. Do not use in code/docs/logs/schema/flags/artifacts.
- `cycle`:
  - Forbidden term. Do not use in code/docs/logs/schema/flags/artifacts.
  - Use `turn` for loop iterations and `part` for progress/accounting.

## Why Parts Are The Source Of Truth

- Parts are the highest-fidelity unit we can observe and count consistently across providers.
- A very capable model can do huge work in one turn; turn-count budgets miss this entirely.
- Part-level indexing gives better recovery and replay granularity than turn-only indexing.
- Artifact and replay contracts are keyed to part indices (`checkout-part`, `part_to_commit`).

## Architecture

```
envoi CLI (packages/cli/envoi_cli/main.py)
  └─ envoi code
       └─ modal run sandbox/modal/deploy.py
            ├─ Orchestrator (packages/code/envoi_code/orchestrator.py)
            ├─ AgentBackend (packages/code/envoi_code/agents/base.py)
            │    ├─ CodexAgent (agents/codex.py) ── runs inside sandbox
            │    └─ OpenCodeAgent (agents/opencode.py) ── runs inside sandbox
            ├─ SandboxBackend (packages/code/envoi_code/sandbox/base.py)
            │    ├─ ModalSandbox (sandbox/modal/backend.py)
            │    └─ E2BSandbox (sandbox/e2b/backend.py)
            ├─ envoi server (localhost:8000) ── test harness from environment/main.py
            └─ MCP server (sandbox/mcp_server.py) ── bridges agent ↔ envoi
```

## Key Files

### SDK (`packages/envoi/envoi/`)
- `environment.py` — `@envoi.suite()`, `@envoi.test()`, `@envoi.setup()` decorators
- `client.py` — `envoi.Client`, `envoi.connect()`, async session API
- `runtime.py` — FastAPI server that hosts an environment
- `logging.py` — shared structured logging context/callback/file sink
- `deploy.py` — Docker-based local deployment
- `constants.py` — Shared constants (ports, timeouts, image names)

### Runner (`packages/code/envoi_code/`)
- `orchestrator.py` — Main orchestrator. Runs inside Modal. Manages agent turns, git checkpoints, trace persistence, and session recovery.
- `models.py` — Pydantic models: `PartRecord`, `TurnRecord`, `AgentTrace`, `SessionEnd`, `EnvoiCall`, `TestingState`, etc.
- `agents/base.py` — `Agent` Protocol. Every agent implements this.
- `sandbox/base.py` — `Sandbox` Protocol + `CommandResult`. Every sandbox implements this.
- `scripts/trace.py` — CLI entrypoint. Launches orchestrator via Modal, handles auto-resume.
- `utils/trace_parquet.py` — Parquet serialization: `agent_trace_to_rows()`, `parquet_to_trace_dict()`.
- `utils/logs_parquet.py` — Structured logs parquet serialization.
- `utils/storage.py` — S3 upload/download for trace and bundle artifacts.
- `utils/git.py` — Git checkpoint operations inside the sandbox.
- `utils/evaluation.py` — Concurrent commit evaluation against envoi.
- `utils/parsing.py` — Parse agent responses into parts and envoi calls.
- `utils/stream.py` — Real-time stream callback for part events.
- `utils/solve.py` — `SolveTracker`: tracks which test paths have been solved.
- `utils/helpers.py` — Small utilities: timestamps, text, tokens, file upload.

### CLI (`packages/cli/envoi_cli/`)
- `main.py` — Unified `envoi` command. `envoi deploy` always available; `envoi code *` available when `envoi-code` is installed.

## Task Loading

`orchestrator.py`'s `load_task(task_dir)` loads a task from a directory path.

- **Canonical**: `task_dir/task.py` with `async def resolve_task(context)` → returns `ResolvedTask`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheSeamau5/envoi](https://github.com/TheSeamau5/envoi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
