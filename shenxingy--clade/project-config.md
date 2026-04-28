---
trigger: always_on
description: - Type: cli + skill-system
---

# Clade — Project Context

## Project Type
- Type: cli + skill-system
- Frontend: N/A (orchestrator has vanilla JS UI but not the primary interface)
- Backend: FastAPI (orchestrator/, port 8000) — optional, CLI layer works standalone
- Test command: cd orchestrator && .venv/bin/python -m pytest tests/ -v
- Verify command: cd orchestrator && python -m py_compile server.py session.py session_tree.py task_queue.py worker.py swarm.py worker_tldr.py worker_review.py worker_utils.py worker_hydrate.py condensers.py config.py github_sync.py ideas.py process_manager.py event_stream.py tracing.py reactions.py routes/tasks.py routes/workers.py routes/webhooks.py routes/ideas.py routes/process.py

## Features (Behavior Anchors)
- install.sh: running `./install.sh` copies skills/hooks/scripts/keybindings to ~/.claude/ without errors
- slt: running `slt` cycles the statusline mode (symbol → percent → number → off)
- /commit: analyzes uncommitted changes, splits into logical commits by module, pushes by default
- /loop: given a goal file, runs supervisor+worker iterations until converged or max-iter
- committer: `committer "type: msg" file1 file2` stages only named files and commits
- loop-runner.sh: runs background loop — supervisor plans tasks, workers execute in parallel via worktrees

## What This Project Is

A two-layer automation toolkit on top of Claude Code CLI:

- **CLI layer** (`configs/`) — skills, hooks, scripts installed via `./install.sh`
- **Orchestrator layer** (`orchestrator/`) — FastAPI web server with worker pool, task queue, GitHub sync, iteration loops

## Key Commands

```bash
# Install CLI layer (skills, hooks, keybindings)
./install.sh

# slt — statusline-toggle (quota pace indicator). See /slt skill.

# Start orchestrator (from project root or orchestrator dir)
cd orchestrator && uvicorn server:app --reload

# Run tests
cd orchestrator && .venv/bin/python -m pytest tests/ -v

# Syntax check (all Python modules)
cd orchestrator && python -m py_compile server.py session.py session_tree.py task_queue.py worker.py swarm.py worker_tldr.py worker_review.py worker_utils.py worker_hydrate.py condensers.py config.py github_sync.py ideas.py process_manager.py event_stream.py tracing.py reactions.py routes/tasks.py routes/workers.py routes/webhooks.py routes/ideas.py routes/process.py

# MCP Server — expose skills as MCP tools for external AI coding tools
# After install.sh, configure in ~/.claude/settings.json:
# { "mcpServers": { "clade": { "command": "python", "args": ["/path/to/orchestrator/mcp_server.py"] } } }
# Then restart Claude Code. Skills appear as `clade_<name>` MCP tools.
```

## Architecture — Two Layers

### CLI Layer (`configs/`)
- `skills/` — skill prompts invoked via `/skill-name` in Claude Code
- `hooks/` — pre/post hooks for Claude Code events
- `scripts/` — shell utilities (e.g., `committer.sh`)
- `keybindings.json` — Claude Code keyboard shortcuts

### Orchestrator Layer (`orchestrator/`)
Key modules (import DAG — leaf → root):

```
config.py            ← leaf: constants, settings, utilities
ideas.py             ← leaf: IdeasManager, async idea CRUD (no project imports)
process_manager.py   ← leaf: ProcessPool, start.sh lifecycle (no project imports)
worker_tldr.py       ← leaf: TLDR generation + scoring (no project imports)
worker_review.py     ← leaf: oracle + PR review (no project imports)
    ↑
github_sync.py       ← gh CLI wrappers (issues, push, sync)
task_queue.py        ← SQLite-backed task CRUD
    ↑
worker.py            ← WorkerPool, SwarmManager
session.py           ← ProjectSession, registry, status_loop
    ↑
server.py            ← FastAPI app, remaining routes, router mounts
mcp_server.py        ← MCP server exposing skills as MCP tools (stdio transport)
routes/tasks.py      ← Task CRUD + bulk-action routes
routes/workers.py    ← Worker control + inspection routes
routes/webhooks.py   ← GitHub webhook handler
routes/ideas.py      ← Ideas API routes (CRUD, evaluate, execute, promote)
routes/process.py    ← Process manager API routes
```

```
config.py            ← leaf: constants, settings, utilities
ideas.py             ← leaf: IdeasManager, async idea CRUD (no project imports)
process_manager.py   ← leaf: ProcessPool, start.sh lifecycle (no project imports)
worker_tldr.py       ← leaf: TLDR generation + scoring (no project imports)
worker_review.py     ← leaf: oracle + PR review (no project imports)
    ↑
github_sync.py       ← gh CLI wrappers (issues, push, sync)
task_queue.py        ← SQLite-backed task CRUD
    ↑
worker.py            ← WorkerPool, SwarmManager
session.py           ← ProjectSession, registry, status_loop
    ↑
server.py            ← FastAPI app, remaining routes, router mounts
routes/tasks.py      ← Task CRUD + bulk-action routes
routes/workers.py    ← Worker control + inspection routes
routes/webhooks.py   ← GitHub webhook handler
routes/ideas.py      ← Ideas API routes (CRUD, evaluate, execute, promote)
routes/process.py    ← Process manager API routes
```

### Key File Map
| File | Purpose |
|------|---------|
| `config.py` | `GLOBAL_SETTINGS`, `_ALLOWED_TASK_COLS`, model aliases, cost utils |
| `task_queue.py` | SQLite CRUD for tasks, loops, messages, interventions |
| `worker.py` | `WorkerPool`, `SwarmManager`, core execution engine |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shenxingy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-17 -->
