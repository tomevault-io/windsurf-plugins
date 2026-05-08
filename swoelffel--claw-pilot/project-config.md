---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

`claw-pilot` v0.81.0 — **CLI + web dashboard** that orchestrates multiple claw-runtime agent instances on a Linux or macOS server. It handles discovery, provisioning, lifecycle management, permanent cross-channel sessions, and extensible middleware pipeline.

All instances use the **claw-runtime** engine — a native Node.js engine (`src/runtime/`), managed via PID file daemon.

Not published on npm — installed from source only (`/opt/claw-pilot`).
GitHub: https://github.com/swoelffel/claw-pilot

## Tech stack

- **Runtime**: Node.js >= 22.12.0, ESM, pnpm
- **CLI**: Commander.js + @inquirer/prompts
- **HTTP/WS**: Hono + ws
- **DB**: better-sqlite3 (SQLite, WAL mode, schema v37)
- **UI**: Lit web components + Vite
- **Build**: tsdown (CLI) + vite (UI)
- **Tests**: Vitest
- **Lint**: oxlint
- **LLM SDK**: Vercel AI SDK `ai` v6.x

## Language standard

All documentation and code comments must be written in **English**.
Commit messages follow conventional commits in English.
Exception: DB migration context and UI localization strings may reference French for historical reasons.

## Contribution workflow

See [CONTRIBUTING.md](CONTRIBUTING.md) for the full contributor guide. The three non-negotiable rules:

1. **Branch off `develop`, PR against `develop`** — never `main`. `main` is reserved for release PRs.
2. **Never bump `package.json` version** in a feature PR — reserved for the release PR (`develop → main`).
3. **Never skip lefthook hooks** (`--no-verify`) — CI enforces the same checks and will reject the PR.

## Key commands

```sh
pnpm build:cli     # Build CLI only (dist/)
pnpm build         # Build CLI + UI
pnpm build:safe    # typecheck:all then build
pnpm test:run      # Run tests once
pnpm test:e2e      # Run e2e tests (real HTTP server, in-memory DB)
pnpm typecheck     # tsc --noEmit
pnpm typecheck:all # tsc --noEmit (backend + UI)
pnpm lint          # oxlint src/
pnpm lint:all      # oxlint src/ + ui/src/
pnpm format        # prettier --write
pnpm format:check  # prettier --check (CI mode)
pnpm spellcheck    # cspell (en + fr dictionaries)
pnpm knip          # Dead code detection
pnpm check:circular # Circular dependency check via madge
```

Run a single test file or case:
```sh
pnpm vitest run src/dashboard/__tests__/routes.test.ts
pnpm vitest run -t "POST /api/instances/:slug/start"
```

Pre-commit hooks (lefthook): format:check + lint:all + typecheck:all.
Pre-push hooks: test:run + spellcheck + no-silent-catches gate. Commits must follow conventional commits (commitlint).

## Architecture

```
src/
  index.ts          # CLI entry point (Commander root)
  commands/         # CLI commands — thin wrappers over core/
  core/             # All business logic
  dashboard/        # HTTP server (Hono) + WebSocket monitor
  db/               # SQLite schema + migrations (schema.ts) — current version: 37
  lib/              # Shared utilities (logger, constants, errors, platform, poll, xdg, shell...)
  runtime/          # claw-runtime engine (bus, provider, session, tool, agent, plugin, middleware, mcp, channel, engine)
  server/           # ServerConnection interface + LocalConnection impl
  wizard/           # Interactive creation wizard (@inquirer/prompts)
ui/
  src/              # Frontend — Lit web components, built to dist/ui/
    components/     # Reusable UI components (cards, dialogs, status badges...)
    services/       # Auth state, WS monitor, router, update poller (extracted from app.ts)
    localization/   # i18n via @lit/localize (6 languages)
    styles/         # Design tokens, shared CSS
templates/          # Workspace bootstrap files + systemd/nginx templates
docs/architecture/  # Functional architecture (split into focused files) — read before major changes
docs/main-doc.md    # Redirect stub → architecture/README.md
```

## Data model (SQLite `~/.claw-pilot/registry.db`)

| Table | Role |
|---|---|
| `servers` | Physical servers (V1: always 1 local row) |
| `instances` | Instances — slug, port, state, config_path, default_named_key_id |
| `agents` | Agents per instance or blueprint (named_key_id override) |
| `ports` | Port reservation registry (anti-conflict) |
| `config` | Global key-value config |
| `events` | Audit log per instance |
| `agent_files` | Workspace files per agent |
| `agent_links` | Agent links (a2a / spawn) |
| `blueprints` | Reusable team templates |
| `agent_blueprints` | Standalone reusable agent templates (id TEXT PK, config_json, category) |
| `agent_blueprint_files` | Workspace files per agent blueprint |
| `named_api_keys` | Encrypted API keys (AES-256-GCM), global scope |
| `rt_sessions` | claw-runtime sessions — permanent (one per agent, cross-channel) or ephemeral. Key: `<slug>:<agentId>` (permanent) or `<slug>:<agentId>:<channel>:<peerId>` (ephemeral) |
| `rt_messages` | Messages per session |
| `rt_parts` | Message parts (text, tool-call, tool-result) |
| `rt_events` | Runtime events (bus events persisted for dashboard) |
| `rt_task_activities` | Task activity timeline — chronological log of all task mutations + comments |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [swoelffel/claw-pilot](https://github.com/swoelffel/claw-pilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
