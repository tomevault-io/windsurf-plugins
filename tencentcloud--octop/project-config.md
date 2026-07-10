---
trigger: always_on
description: Navigation guide for AI coding agents working in this repository.
---

# AGENTS.md

Navigation guide for AI coding agents working in this repository.

## 1. Collaboration principles

> Favor caution over speed; trivial tasks may relax these rules. These principles complement [§10 Change workflow](#10-change-workflow) and [§11 Communication](#11-communication).

### Think before writing

- State assumptions up front; ask when unsure — do not guess.
- When multiple interpretations exist, list them and let the user choose — do not pick silently.
- Suggest simpler approaches when they exist; push back when appropriate.
- Stop when blocked; name exactly what is unclear.

### Simplicity first

- Write the minimum code that solves the problem; no unrequested features, abstractions, or config knobs.
- Do not add defensive error handling for scenarios that cannot realistically happen.
- Trim the diff when it grows unnecessarily large.

### Surgical edits

- Touch only lines directly related to the task; do not opportunistically "clean up" nearby code, comments, or formatting.
- Do not refactor working code or unify style just because it differs from yours.
- Unrelated dead code: mention it, do not delete it proactively.
- Remove orphan imports, variables, and functions **you** introduced.

### Verifiable outcomes

- Turn tasks into verifiable goals (what to test, which command proves success).
- For multi-step work, sketch a short plan: `step → verify: …`
- Before saying "done", provide verification evidence; the default ship bar is **`make all` green** (see [§6 Run commands](#6-run-commands), [§10 Change workflow](#10-change-workflow)).

## 2. What this is

**Octop** — self-hosted AI assistant platform (multi-user, multi-agent).
One Python wheel: FastAPI backend + React dashboard + Click CLI.
No external queue. No required services beyond an LLM provider.

## 3. Tech stack

| Layer | Technology |
|-------|-----------|
| Language | Python 3.11+ |
| Web framework | FastAPI + uvicorn |
| Async runtime | asyncio (no threads except `run_in_executor`) |
| Database | SQLite via `aiosqlite` (WAL mode) |
| LLM runtime | `harness-agent` (LangGraph) at `/workspace/harness-agent` |
| Gateway | `harness-gateway` at `/workspace/harness-gateway` |
| Frontend | React 18 + TypeScript + Vite |
| Package manager | uv — always `uv run pytest`, never bare `pytest` |
| API docs UI | Scalar (`scalar-fastapi`) at `/api/docs` |

## 4. Package layout

High-level tree — see [§5 Module boundaries](#5-module-boundaries) for what each folder owns and what it may import.

```
Octop (`octop/` workspace directory)     Python package root (`src/octop/`)
  config.py
  launch.py                   composition root: OctopServer + FastAPI + uvicorn
  i18n/                       locale JSON bundles + tr() + domain helpers
  infra/                      domain core (agents, DB, gateway, …)
  api/                        HTTP adapters (FastAPI)
  cli/                        Click commands
  dashboard/                  built SPA artifact — do NOT edit

dashboard/                    frontend source (Vite) — edit here
docs/                         human-written reference (e.g. `api.md`)
tests/                        pytest (`unit/`, `integration/`)
```

## 5. Module boundaries

**Rule of thumb:** dependencies flow **inward** — transport layers call domain; domain never calls HTTP/CLI. Within `infra/`, leaf modules (`utils/`, `db/repos/`, `errors.py`) stay free of higher-level orchestration.

### Layer overview

```
dashboard/ ──HTTP──► api/ ──► infra/ ──► infra/utils/, octop.config
cli/ ──► launch.py ──► api/ + infra/
```

| Layer | Role | May import | Must NOT import |
|-------|------|------------|-----------------|
| `octop.config` | Env-based `OctopConfig` | stdlib, pydantic | `infra/`, `api/`, `cli/`, `launch.py` |
| `octop.i18n` | Locale JSON + `tr()` + per-namespace helpers | `infra/utils/locale`, stdlib | `api/`, `cli/`, `dashboard/` |
| `octop.launch` | Wire `OctopServer`, `build_app`, uvicorn for `octop run` | `infra/`, `api/` | business logic; must not be imported by `infra/` |
| `infra/utils/` | Pure helpers (paths, ulid, env files, Ollama) | stdlib, third-party | any other `infra/*` domain code |
| `infra/db/repos/` | One repo per table — SQL only | `infra/db/_base`, `infra/utils/` | `agents/`, `gateway/`, `api/`, orchestration |
| `infra/` (domain) | Business logic & orchestration | `infra/utils/`, `infra/db/`, `octop.config`, peer `infra/*` subpackages, `infra/errors`, `infra/metrics` | `api/`, `cli/`, `launch.py`, `dashboard/` |
| `api/` | HTTP: routing, auth, SSE, OpenAPI | `infra/`, `octop.config`, sibling `api/*` | `cli/`, `launch.py`; no business rules that belong in `infra/` |
| `cli/` | Terminal UX | `infra/`, `octop.config`, `launch.py`, sibling `cli/*` | `api/`; domain logic duplicated from `infra/` |
| `dashboard/` | React UI | `dashboard/src/api` → backend over HTTP | Python packages; no direct DB or `infra/` access |

**Hard bans**

- `infra/` → `api/`, `cli/`, or `launch.py`.
- `api/` → `cli/` or `launch.py`.
- `cli/` → `api/` (use `launch.py` for `octop run` instead).
- `infra/db/repos/` → any non-DB `infra` package.
- `infra/utils/` → any non-utils `infra` package.
- Routers/helpers in `api/routers/` must stay thin: validate HTTP, call `infra/`, map errors — not new domain rules.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TencentCloud/Octop](https://github.com/TencentCloud/Octop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-10 -->
