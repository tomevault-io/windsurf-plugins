---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What Shipwright is

Shipwright Harness is **the open-source (MIT) autonomous delivery agent for Claude Code** — a deployable agent plus the autonomous coding system that powers it. The system is a Claude Code plugin (`shipwright`) covering the full delivery loop — **spec → plan → execute → review → deploy** — alongside a metrics dashboard and a Shipwright agent. The brand is **Shipwright Harness**; the installable package is **`shipwright`**.

> The plugin is repo-agnostic: it runs its planning/execution/review/deploy commands against *any* repository. This repo is both the source of the plugin **and** the codebase it ships against.

## Architecture — four artifacts, sequenced A → B → C → D

| Phase | Artifact | Directory | What it is |
|---|---|---|---|
| **A** | **Plugin** (the system) | `plugins/shipwright/` | Commands, skills, agents, scripts users `/plugin install`. Repo-agnostic. |
| **B** | **Metrics dashboard** | `metrics/` | Stateless Hono service: task-store-backed JSON endpoints + a server-rendered dashboard. No database. |
| **C** | **Shipwright agent** | `agent/` | Hono service + Prisma store; a thin autonomous runner: pick next ready task → build → ship PR → forward metrics. |
| **D** | **Task store service** | `task-store/` | Postgres-backed task queue, PR tracking, and scoped tokens. Prisma schema defines `Task`, `PullRequest`, and `TaskToken` models; re-exported as `@shipwright/task-store`. Replaces the JSON file fallback. |

Supporting surfaces (not phased):
- `site/` — Astro + Tailwind marketing site (**shipwrightharness.com**). Self-contained; **not** a Bun workspace; Playwright smoke tests.
- `brand/` — locked design system (`BRAND.md`, `tokens.json`) + CSS build + lint, consumed by `site/` and brand artifacts. Editing brand artifacts triggers the `shipwright-brand` skill.
- `state/` — git-ignored local JSON task-store fallback + cached review state (only written when the GitHub backend isn't active).

## Commands

go-task (`Taskfile.yml`) is the single local entrypoint; the root `package.json` mirrors a subset as `bun run` scripts.

```bash
task setup        # bun install across all workspaces
task ci           # lint → check-strings → typecheck → check-config-docs → check-version-sync → test:coverage → secret-scan (the merge-blocking gate; CI runs this exact chain)
task test         # bun test            (single file: bun test path/to/file.test.ts)
task test:coverage  # bun test --coverage --coverage-reporter=lcov, then gate on aggregate 80/80 lines/functions (scripts/check-coverage.ts)
task lint         # bunx biome lint .
task format       # bunx biome format --write .
task typecheck    # bun run --filter='*' --sequential typecheck
task check-strings  # scan entire repo for banned/confidential identifiers (client names, internal infra IDs)
```

Database (admin service):
```bash
export DATABASE_URL_SHIPWRIGHT_ADMIN="postgresql://user:password@localhost:5432/shipwright_admin"
task db:provision   # prisma migrate deploy (idempotent)
task db:migrate     # prisma migrate dev (creates a new migration)
```

Marketing site (run from `site/`, **not** part of `bun test`):
```bash
cd site && npm run dev      # astro dev
cd site && npm run build    # astro build
cd site && npm test         # playwright test (*.spec.ts)
```

> `bunfig.toml` excludes `site/**` from the root `bun test` scan — Playwright's `*.spec.ts` files would otherwise crash Bun's runner. Keep site tests as `*.spec.ts` to stay isolated.

Run the metrics service locally:

```bash
task api        # start metrics dashboard in offline mode → http://localhost:3460/dashboard
task ui         # same as task api (API and UI are one process)
task dev        # dev supervisor: starts metrics + Ctrl-C kills all children
task stack      # full dev stack in a tmux session (6 panes) — requires tmux + Docker + state/dev-agent.env
task hitl       # human-in-the-loop runner: boots task-store (:3002) + admin (:3001), then loops — pulls the next ready task and launches Claude Code with the right command
```

**`task stack` needs `state/dev-agent.env`** — if it's missing, the first `task stack` run auto-creates it from `state/dev-agent.env.example` and exits; fill in `CLAUDE_CODE_OAUTH_TOKEN` (or `ANTHROPIC_API_KEY`) in the newly-created file, then re-run `task stack`. See `docs/quickstart.md` for the full flow.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [app-vitals/shipwright](https://github.com/app-vitals/shipwright) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
