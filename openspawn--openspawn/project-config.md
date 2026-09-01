---
trigger: always_on
description: Quick reference for AI agents working on OpenSpawn. Start here.
---

# AGENTS.md — AI Agent Quick Reference

Quick reference for AI agents working on OpenSpawn. Start here.

## What Is This?

Multi-agent coordination platform. Agents get tasks, earn credits, communicate. Humans monitor via dashboard.

**Stack**: FastAPI (Python) + React Dashboard + PostgreSQL (prod) / SQLite (local) + MCP (served by API)
**Package Manager**: pnpm (frontend) + uv (Python API)
**Build System**: Nx

---

## Project Structure

```
apps/
  dashboard/       -> React dashboard (openspawn.ai + bikinibottom.ai via env vars)
  team/            -> Internal team dashboard
  website/         -> openspawn.ai marketing site + docs (MDX)
  platform/        -> openspawn.ai landing page
  api/             -> FastAPI backend (REST + OpenAPI + MCP) — Python, uv
  sandbox-cli/     -> CLI entry point for sandbox
libs/
  dashboard-data/  -> Shared hooks, auth, utilities
  dashboard-ui/    -> Shared React UI components
  docs-ui/         -> Shared doc components (layout, callout, MDX, search)
  design-tokens/   -> Design system (colors, spacing, typography)
  demo-data/       -> Simulation engine, scenarios, fixtures
  shared-types/    -> Shared TypeScript types and enums
  test-utils/      -> Shared test utilities

tools/
  sandbox/         -> Coordination sandbox server (SSE + MCP + A2A)

packages/
  openspawn/       -> npm CLI package (init + preview + start), published to npmjs.com
  coordinator/     -> Coordination server package
```

---

## Commands

```bash
pnpm install                                              # Install
cd apps/api && uv run uvicorn app.main:app --reload       # Dev (FastAPI)
pnpm run dev:sandbox                                      # Dev (Sandbox + Dashboard)
npx openspawn init                                        # CLI — scaffold project
npx openspawn preview                                     # CLI — preview org in local sandbox
npx openspawn start                                       # CLI — start coordinator
pnpm exec nx run-many -t build                            # Build
pnpm exec nx test dashboard                                # Unit tests
pnpm exec nx e2e demo-e2e                                  # E2E tests
pnpm exec nx run-many -t lint                             # Lint
pnpm exec oxfmt --write .                                 # Format
cd apps/api && uv run alembic upgrade head                # Database migrations
```

---

## Key URLs

- **Demo**: https://bikinibottom.ai
- **API**: https://openspawn.ai/api/ | docs: https://openspawn.ai/api/docs
- **Website**: https://openspawn.ai
- **Docs**: https://openspawn.ai/docs/ (docs.openspawn.ai redirects here)
- **Dev dashboard**: http://localhost:4200 | demo mode: http://localhost:4200/?demo=true
- **Dev sandbox**: http://localhost:3333
- **Dev API**: http://localhost:8000 | docs: http://localhost:8000/docs

---

## Architecture (1-minute version)

1. **Sandbox server** (`tools/sandbox/`) hosts the REST/SSE API and serves pre-built dashboards
2. **Dashboard** (`apps/dashboard/`) serves both openspawn.ai and bikinibottom.ai (via `VITE_DASHBOARD_THEME` + `VITE_SANDBOX_MODE` env vars)
3. **API** (`apps/api/`) manages tasks, credits, messages, MCP tools, and agent spawning (FastAPI + SQLAlchemy)
4. **Preview mode** — `openspawn preview` launches the sandbox simulation + dashboard at `localhost:3333` (deterministic, zero LLM calls)
5. **Agent spawning** — `openspawn start` launches the Python coordinator which spawns Claude Code CLI subprocesses with a configurable concurrency cap
6. **Two-tier model** — Tier 1 (local): SQLite + asyncio scheduler, no Docker needed. Tier 2 (deployed): PostgreSQL + arq/Redis + Docker
7. **Demo mode** simulates everything client-side (no backend needed) via `libs/demo-data/`
8. **Docker** builds dashboard + team + website, serves all via sandbox server on VPS (Tier 2 only)

Full details: [ARCHITECTURE.md](ARCHITECTURE.md)

---

## Database Entities

| Entity              | Purpose                                                                      |
| ------------------- | ---------------------------------------------------------------------------- |
| `Agent`             | AI agents with levels (L1-L10), parent hierarchy, balance, autonomy level    |
| `AgentCapability`   | Skills per agent with proficiency level                                      |
| `Task`              | Work items with Kanban status flow, optional per-task autonomy override      |
| `TaskDependency`    | Blocking relationships between tasks                                         |
| `CreditTransaction` | Debits/credits with audit trail                                              |
| `Channel`           | Communication channels (task, DM, broadcast)                                 |
| `Message`           | Messages in channels                                                         |
| `Event`             | Append-only system audit log                                                 |
| `EventSubscription` | Agent subscriptions to typed coordination events (wildcard pattern matching) |
| `ApprovalRequest`   | Gated actions awaiting human/manager approval (risk vs autonomy)             |
| `Artifact`          | Versioned work products (components, tests, schemas) with approval workflow  |

---

## Conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openspawn/openspawn](https://github.com/openspawn/openspawn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
