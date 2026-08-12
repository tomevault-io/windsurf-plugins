---
trigger: always_on
description: Open-source dashboard for AI agent orchestration. Manage agent fleets, track tasks, monitor costs, and orchestrate workflows.
---

# Paddock

Open-source dashboard for AI agent orchestration. Manage agent fleets, track tasks, monitor costs, and orchestrate workflows.

**Stack**: Next.js 16, React 19, TypeScript 5, SQLite (better-sqlite3), Tailwind CSS 3, Zustand, pnpm

## OpenClaw Node Deployment Notes

- These notes apply to operator-managed Paddock worktrees: `<live-worktree>` (live `main`) and `<dev-worktree>` (dev branch).
- Paddock should run from `racecraft-lab/Paddock` `main`.
- Active systemd unit: `paddock.service`
- Active startup wrapper: `~/.local/bin/mc-start.sh`
- The wrapper resolves runtime secrets from the operator's configured secret manager at startup.
- Active service worktree: `<live-worktree>` on `main`; `<dev-worktree>` is the development worktree on the active feature branch.
- OpenClaw is a separate deploy surface on the operator node. The gateway should run from `<openclaw-release-symlink>`, which should point at the clean tagged release tree, not from a Homebrew global package path.
- If you change startup assumptions, verify both:
  - `systemctl --user status --no-pager paddock.service`
  - `systemctl --user status --no-pager openclaw-gateway.service`

## Prerequisites

- Node.js >= 22 (LTS recommended; 24.x also supported)
- pnpm (`corepack enable` to auto-install)

## Setup

```bash
pnpm install
pnpm build
```

Secrets (AUTH_SECRET, API_KEY) auto-generate on first run if not set.
Visit `http://localhost:3000/setup` to create an admin account, or set `AUTH_USER`/`AUTH_PASS` in `.env` for headless/CI seeding.

## Run

```bash
pnpm dev              # development (localhost:3000)
pnpm start            # production
pnpm start:standalone # standalone mode (after build)
```

## Docker

```bash
docker compose up                 # zero-config
bash install.sh --docker          # full guided setup
```

Production hardening: `docker compose -f docker-compose.yml -f docker-compose.hardened.yml up -d`

## Tests

```bash
pnpm test             # unit tests (vitest)
pnpm test:e2e         # end-to-end (playwright)
pnpm typecheck        # tsc --noEmit
pnpm lint             # eslint
pnpm test:all         # lint + typecheck + test + build + e2e
```

Codex sandbox note: run `pnpm test` outside the sandbox. The suite uses local
runtime resources that can fail under sandboxed execution.

## Key Directories

```
src/app/          Next.js pages + API routes (App Router)
src/components/   UI panels and shared components
src/lib/          Core logic, database, utilities
.data/            SQLite database + runtime state (gitignored)
scripts/          Install, deploy, diagnostics scripts
docs/             Documentation and guides
```

Path alias: `@/*` maps to `./src/*`

## Repo Knowledge Map

- Canonical machine-readable index: `docs/ai/repo-knowledge-index.json`
  with schema at `docs/ai/repo-knowledge-index.schema.json`.
- Durable intent: `docs/rc-factory-v1-prd.md` and
  `docs/ai/rc-factory-technical-roadmap.md`.
- Current SpecKit ledgers and status pointers: `docs/ai/specs/`,
  `docs/ai/specs/SPEC-012A-workflow.md`, and
  `docs/ai/specs/autopilot-state.json`.
- Completed SPEC-012B workflow and generated artifacts:
  `docs/ai/specs/.process/SPEC-012B-workflow.md` and
  `specs/012b-harness-gardening-guards/`.
- QA and recovery evidence: `docs/qa/pilot-smoke-checklist.md` and
  `docs/runbook/migration-rollback.md`.
- Workflow contract source: `docs/ai/workflows/paddock/workflow-contract.yaml`.
- Local checks: `pnpm knowledge:index:check`, `pnpm knowledge:index:smoke`,
  and `pnpm guardrails -- --suite repo-knowledge-index`.
- GitNexus refresh guidance stays in the GitNexus section below. `.gitnexus/`
  remains ignored local output and is not CI truth.

## Data Directory

Set `PADDOCK_DATA_DIR` env var to change the data location (defaults to `.data/`).
Database path: defaults to `<PADDOCK_DATA_DIR>/paddock.db`.

## Conventions

- **Commits**: Conventional Commits (`feat:`, `fix:`, `docs:`, `test:`, `refactor:`, `chore:`)
- **No AI attribution**: Never add `Co-Authored-By` or similar trailers to commits
- **Package manager**: pnpm only (no npm/yarn)
- **Icons**: No icon libraries -- use raw text/emoji in components
- **Standalone output**: `next.config.js` sets `output: 'standalone'`

## Agent Control Interfaces

Paddock provides three interfaces for autonomous agents:

### MCP Server (recommended for agents)
```bash
# Add to any Claude Code agent:
claude mcp add paddock -- node /path/to/Paddock/scripts/mc-mcp-server.cjs

# Environment config:
MC_URL=http://127.0.0.1:3000 MC_API_KEY=<key>
```
49 tools: agents, tasks, sessions, memory, soul, comments, tokens, skills, cron, status, runs, knowledge, and evals.
See `docs/cli-agent-control.md` for full tool list.

### CLI
```bash
pnpm mc agents list --json
pnpm mc tasks queue --agent Aegis --max-capacity 2 --json
pnpm mc events watch --types agent,task
```

### REST API
OpenAPI spec: `openapi.json`. Interactive docs at `/docs` when running.

## Common Pitfalls

- **Standalone mode**: Use `pnpm start:standalone`, not `pnpm start` (which requires full `node_modules`)
- **better-sqlite3**: Native addon -- needs rebuild when switching Node versions (`pnpm rebuild better-sqlite3`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [racecraft-lab/Paddock](https://github.com/racecraft-lab/Paddock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
