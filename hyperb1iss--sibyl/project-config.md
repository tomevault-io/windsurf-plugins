---
trigger: always_on
description: **Sibyl** is a graph-backed memory system - an MCP server and web app providing persistent memory,
---

# Sibyl Development Guide

## Project Overview

**Sibyl** is a graph-backed memory system - an MCP server and web app providing persistent memory,
search, and task coordination through a Graphiti-powered knowledge graph.

**See package READMEs for detailed documentation:**

- [`README.md`](README.md) — Project overview, quickstart, philosophy
- [`apps/api/README.md`](apps/api/README.md) — Server daemon (sibyld), MCP API, REST endpoints
- [`apps/cli/README.md`](apps/cli/README.md) — Client CLI (sibyl), user commands
- [`apps/web/README.md`](apps/web/README.md) — Web UI, components, React Query hooks
- [`packages/python/sibyl-core/README.md`](packages/python/sibyl-core/README.md) — Core library,
  models, graph client

---

## Sibyl Integration

**This project uses Sibyl as its own knowledge repository.**

### ALWAYS Use Skills

**Use `/sibyl`** for ALL Sibyl operations. This skill knows the correct patterns and handles
authentication properly.

- `/sibyl` - Search, explore, add knowledge, manage tasks, project audits, sprint planning

**Never call Sibyl MCP tools or CLI directly** without going through a skill first.

**Use `/uv`** before running any `uv` commands. The skill provides current best practices and
prevents common mistakes.

- **NEVER run `uv pip`** — it bypasses the project's dependency management. Use `uv add`, `uv sync`,
  or `uv run` instead.

### Research → Do → Reflect Cycle

Every significant task follows this cycle:

**1. RESEARCH** (before coding)

```
/sibyl search "topic"
/sibyl explore patterns
```

**2. DO** (while coding)

```
/sibyl task start <id>
```

**3. REFLECT** (after completing)

```
/sibyl task complete <id> --learnings "What I learned"
/sibyl add "Pattern Title" "What, why, how, caveats"
```

---

## Quick Reference

### Monorepo Structure

```
sibyl/
├── apps/
│   ├── api/              # sibyld - Server daemon (serve, worker, db)
│   ├── cli/              # sibyl - Client CLI (task, search, add, etc.)
│   └── web/              # Next.js 16 frontend
├── packages/python/
│   └── sibyl-core/       # Shared library (models, graph, tools)
├── skills/               # Claude Code skills
└── charts/               # Helm charts
```

### CLI Executables

| Binary   | Package    | Purpose                                    |
| -------- | ---------- | ------------------------------------------ |
| `sibyld` | `apps/api` | Server daemon (serve, worker, db, up/down) |
| `sibyl`  | `apps/cli` | Client CLI (task, search, add, explore)    |

### Development Commands

**⚡ Always use `moon` for monorepo operations.** Moon handles task orchestration, caching, and
cross-package dependencies. Never use raw `pnpm`/`uv` commands for lint, test, build, or typecheck.

```bash
# Lifecycle
moon run dev              # Start everything (FalkorDB, API, worker, web)
moon run stop             # Stop all services

# Quality (from any directory)
moon run :lint            # Lint current project (or all if at root)
moon run :test            # Test current project
moon run :typecheck       # Typecheck current project
moon run :check           # All quality checks (lint + typecheck + test)

# Target specific packages
moon run web:lint         # Lint web app
moon run api:test         # Test API
moon run core:check       # Full check on sibyl-core

# Build & Install
moon run install-dev      # Install everything editable (sibyl, sibyld, skills)
moon run install          # Install everything (production)
```

**Why moon?** Caches results, runs only what changed, handles dependencies between packages. A bare
`pnpm lint` won't respect the monorepo graph and may miss cross-package issues.

### Dev Introspection Tools

**Use these when debugging Sibyl itself.** Requires OWNER role.

```bash
# System health at a glance
sibyl debug status              # API/worker/graph/queue health + recent errors

# Inspect the graph directly
sibyl debug schema              # Entity types and counts
sibyl debug query "MATCH ..."   # Run read-only Cypher queries

# Server logs
sibyl logs tail                 # Last 50 log entries
sibyl logs tail -n 100          # More entries
sibyl logs tail -l error        # Filter by level (debug/info/warning/error)
sibyl logs tail -s api          # Filter by service (api/worker)
sibyl logs tail -f              # Stream in real-time (Ctrl+C to stop)

# JSON output for scripting
sibyl debug status --json
sibyl logs tail --json
```

**When to use:**

- Tests failing mysteriously → `sibyl logs tail -l error`
- Graph queries returning unexpected results → `sibyl debug query "MATCH ..."`
- Need to understand entity distribution → `sibyl debug schema`
- Something feels broken → `sibyl debug status`

### Ports

| Service   | Port |
| --------- | ---- |
| API + MCP | 3334 |
| Frontend  | 3337 |
| FalkorDB  | 6380 |

---

## Key Patterns

### Multi-Tenancy

**Every graph operation requires org context - NO defaults:**

```python
manager = EntityManager(client, group_id=str(org.id))
```

Each organization gets its own isolated FalkorDB graph. Forgetting org scope queries the wrong graph
or breaks isolation.

### FalkorDB Write Concurrency

FalkorDB's `BlockingConnectionPool` (50 connections, 60s timeout) handles write concurrency

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hyperb1iss/sibyl](https://github.com/hyperb1iss/sibyl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
