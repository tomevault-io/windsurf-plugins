---
trigger: always_on
description: NOTE: Do NOT add "Co-Authored-by" trailers mentioning AI models (e.g., Opus, Claude) in commit messages PRs, or comments.
---

# CLAUDE.md — Valet Development Guide

NOTE: Do NOT add "Co-Authored-by" trailers mentioning AI models (e.g., Opus, Claude) in commit messages PRs, or comments.

## What This Project Is

Valet is a hosted background coding agent platform. Users interact with an AI coding agent through a web UI, Slack, or Telegram. Each session runs in an isolated Modal sandbox with a full dev environment (VS Code, browser via VNC, terminal, and an OpenCode agent with 73 custom tools). A per-user orchestrator ("Jarvis") manages sessions, routes messages across channels, and maintains long-term memory. The architecture is modeled after Ramp's Inspect system.

## Project Structure

```
valet/
├── packages/
│   ├── client/              # React SPA (Vite + TanStack Router + Query + Zustand)
│   ├── worker/              # Cloudflare Worker (Hono + D1 + R2 + Durable Objects)
│   ├── shared/              # Shared TypeScript types & errors
│   ├── runner/              # Bun/TS runner for inside sandboxes
│   ├── sdk/                 # Integration & channel SDK contracts, MCP client, UI components
│   ├── plugin-github/       # GitHub integration (actions: PRs, issues, webhooks)
│   ├── plugin-slack/        # Slack (actions + channel adapter)
│   ├── plugin-gmail/        # Gmail integration
│   ├── plugin-google-*/     # Google Calendar, Drive, Sheets integrations
│   ├── plugin-linear/       # Linear issue tracking
│   ├── plugin-notion/       # Notion integration
│   ├── plugin-stripe/       # Stripe integration
│   ├── plugin-cloudflare/   # Cloudflare API integration
│   ├── plugin-sentry/       # Sentry error tracking
│   ├── plugin-deepwiki/     # DeepWiki knowledge base
│   ├── plugin-telegram/     # Telegram (channel adapter)
│   ├── plugin-browser/      # Browser skill (content-only)
│   ├── plugin-workflows/    # Workflow skill (content-only)
│   ├── plugin-sandbox-tunnels/  # Tunnel skill (content-only)
│   └── plugin-memory-compaction/ # Memory compaction tool (content-only)
├── backend/                 # Modal Python backend
├── docker/
│   ├── Dockerfile.sandbox   # Sandbox container image
│   ├── start.sh             # Sandbox startup script
│   └── opencode/            # OpenCode config: tools/
├── docs/
│   └── specs/               # Subsystem specs (source of truth per domain)
├── V1.md                    # Original architecture spec (may be outdated)
├── V2.md                    # Orchestration layer spec (orchestrators, channels, personas)
├── Makefile                 # Dev, test, deploy commands
├── docker-compose.yml       # Local dev (OpenCode container)
└── .beans/                  # Task tracking (beans)
```

## Tech Stack Quick Reference

| Layer | Tech | Key Files |
|-------|------|-----------|
| Frontend | React 19, Vite 6, TanStack Router/Query, Zustand, Tailwind, Radix UI | `packages/client/src/` |
| Worker | Cloudflare Workers, Hono 4, D1 (SQLite via Drizzle ORM), R2, Durable Objects | `packages/worker/src/` |
| Shared | TypeScript types, error classes, scope keys | `packages/shared/src/` |
| SDK | Integration contracts, channel contracts, MCP client/OAuth, UI components | `packages/sdk/src/` |
| Runner | Bun, TypeScript, `@opencode-ai/sdk`, Hono gateway | `packages/runner/src/` |
| Backend | Python 3.12, Modal SDK | `backend/` |
| Sandbox | OpenCode serve (73 tools, 3 skills, 1 plugin), code-server, Xvfb+VNC, TTYD | `docker/` |

## Subsystem Specs

Detailed per-subsystem specifications live in `docs/specs/`. These are the source of truth for each domain's behavior, boundaries, data model, and contracts. When modifying a subsystem, update its spec in the same commit.

| Spec | Covers |
|------|--------|
| [`docs/specs/sessions.md`](docs/specs/sessions.md) | Session lifecycle, state machine, sandbox orchestration, prompt queue, message streaming, hibernation/restore, access control, multiplayer |
| [`docs/specs/sandbox-runtime.md`](docs/specs/sandbox-runtime.md) | Sandbox boot sequence, service ports, auth gateway, Runner process, OpenCode lifecycle, Runner↔DO WebSocket protocol, Modal backend |
| [`docs/specs/real-time.md`](docs/specs/real-time.md) | SessionAgentDO WebSocket handling, EventBusDO, event types, V2 streaming protocol, client reconnection, message deduplication |
| [`docs/specs/workflows.md`](docs/specs/workflows.md) | Workflow definitions, trigger types (webhook/schedule/manual), execution lifecycle, WorkflowExecutorDO, step engine, approval gates, proposals, version history |
| [`docs/specs/auth-access.md`](docs/specs/auth-access.md) | OAuth flows (GitHub/Google), token auth, admin middleware, org model (settings/invites/LLM keys), session access control, JWT issuance |
| [`docs/specs/orchestrator.md`](docs/specs/orchestrator.md) | Orchestrator identity, auto-restart, child session spawning, memory system (FTS), mailbox, channel routing, task board |
| [`docs/specs/integrations.md`](docs/specs/integrations.md) | Integration framework, GitHub (OAuth/webhooks/API proxy), Telegram bot, Gmail, Google Calendar, channel bindings, custom LLM providers, credential storage |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yourbuddyconner) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
