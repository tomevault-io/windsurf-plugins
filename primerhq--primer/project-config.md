---
trigger: always_on
description: You are an AI agent working ON the primer codebase. This file is your contract
---

# AGENTS.md - Contributing to primer (for AI agents)

You are an AI agent working ON the primer codebase. This file is your contract
for making changes. (If instead you were connected to a running primer
deployment over MCP and want to USE it, read
[skills/using-primer-over-mcp.md](skills/using-primer-over-mcp.md).)

---

## 1. The working model: coordinator + parallel subagents

Development on this repo uses a coordinator/worker split:

- The **main session is a coordinator**. It does design, planning, dispatch,
  verification, and merge. It does NOT implement tasks directly.
- Each task gets its **own branch off `main`** and its **own git worktree**, so
  multiple workstreams proceed in parallel without colliding.
- A **subagent implements the task** inside that worktree.
- When the subagent reports done, the coordinator **critically verifies**
  completeness and quality against the Definition of Done (section 4). If it
  passes, the coordinator **merges the branch into `main`**. If not, the
  coordinator **re-dispatches the subagent** with specific fix instructions and
  re-verifies. Reviews are not rubber stamps: read the actual diff, run the
  tests, and reject work that masks a problem instead of fixing it.

### Per-task git worktree flow

```bash
# create the task branch + an isolated checkout (sibling dir)
git worktree add -b feat/<task-slug> ../primer-<task-slug> main
# ... subagent works and commits inside ../primer-<task-slug> ...
# after verification, from the main checkout:
git merge --no-ff feat/<task-slug>
git worktree remove ../primer-<task-slug>
git branch -d feat/<task-slug>
```

The Agent tool's `isolation: "worktree"` option automates the create/cleanup of
the per-task worktree; prefer it when dispatching an implementer.

Constraints that always hold: branch off `main`, never force-push `main`,
conventional commit messages with NO `Co-Authored-By` footer, stage only the
files a task touches (never `git add -A`).

---

## 2. Project setup and structure

- **Stack:** Python 3.12, `uv`, FastAPI, asyncio, Postgres + pgvector, a
  vanilla-React (JSX, no build step) console.
- **Setup:** `uv sync --all-extras` (the optional backends - huggingface,
  docling, lance, channels, docker, kubernetes - live behind extras; the dev
  and test env needs them all); Postgres via `docker compose up -d postgres`; run with
  `uv run primer api` (starts the API plus an in-process worker). A dogfood
  instance is expected to stay healthy on `:9000` between tasks.
- **Layout:**
  - `primer/<subsystem>/` - the backend, one package per subsystem: `api`,
    `model`, `agent`, `chat`, `session`, `worker`, `claim`, `graph`,
    `workspace`, `channel`, `toolset`, `llm`, `embedder`, `vector`, `trigger`,
    `bus`, `mcp`, `harness`, `storage`, `bootstrap`, and more.
  - `ui/` - the operator console (JSX components, `window`-exported pages,
    a hash router).
  - `primectl/` - the CLI client over the REST API.
  - `tests/` - `tests/<subsystem>/` unit tests, `tests/e2e/` end-to-end against
    a live server, `tests/ui_e2e/` Playwright, `tests/distributed/`,
    `tests/docs/` hygiene.
  - `docs/dev/` - the authoritative developer reference (architecture +
    subsystems). `docs/agents/` - agent-usage docs served as internal AI docs
    (ingested into the `_internal_ai_docs` collection; see
    `docs/agents/_README.md`). The operator-facing console docs now live in a
    separate external repo (the `primerhq.github.io` Pages site), not in this
    tree.
  - `skills/` - guidance for agents using a primer deployment (not contribution).

---

## 3. Read before you change anything

Start at [docs/dev/README.md](docs/dev/README.md) for the doc-set map and
subsystem dependency graph, then read
[docs/dev/CONTRIBUTING.md](docs/dev/CONTRIBUTING.md) (required reading order,
PR conventions, common pitfalls).

Before changing a subsystem, read the cross-cutting patterns in
[docs/dev/architecture/](docs/dev/architecture/) that it touches:

- [provider-pattern](docs/dev/architecture/provider-pattern.md) - the
  discriminated-config + registry + factory shape used by every provider.
- [storage](docs/dev/architecture/storage.md) - the `Storage[T]` abstraction,
  lazy per-model tables, JSONB layout, and the query model.
- [rest-api](docs/dev/architecture/rest-api.md) - `make_crud_router`, the six
  CRUD ops, and the RFC7807 `ProblemDetails` error envelope every path returns.
- [claim-machine](docs/dev/architecture/claim-machine.md) and
  [worker-system](docs/dev/architecture/worker-system.md) - leases, claiming,
  the park/resume model.
- [observability](docs/dev/architecture/observability.md) and
  [auto-bootstrap](docs/dev/architecture/auto-bootstrap.md).

Then read the matching `docs/dev/subsystems/<name>.md` for the feature you are
working on.

---

## 4. Definition of Done

A change is NOT complete until every applicable track below is done. The
coordinator verifies each before merging. The detailed how-to for each track is
in [docs/dev/CONTRIBUTING.md](docs/dev/CONTRIBUTING.md) section 2; this is the
checklist.

1. **Backend** - models in `primer/model/`, storage wiring, REST routes under
   `primer/api/routers/` following the rest-api conventions, RFC7807 errors,
   observability hooks.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [primerhq/primer](https://github.com/primerhq/primer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
