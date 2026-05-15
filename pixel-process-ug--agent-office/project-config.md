---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Agent Office is a pixel-art virtual office for AI agent teams. Globally installable CLI tool (`npm i -g agent-office && office`). Agents have live presence states, sit in themed rooms, support multi-LLM providers (Claude Code, OpenAI, Ollama), and can be managed through a retro-styled UI, Telegram bot, or CLI.

## Commands

```bash
# CLI (global install)
office                # Start server + open browser in current project dir
office init           # Interactive setup wizard
office status         # Show running state from PID file
office agents         # List agents from DB
office stop           # SIGTERM via PID file
office --port 8080    # Override port
office --no-open      # Skip browser open (Docker/CI)

# Development
npm run dev           # Start Vite dev server (http://localhost:4173)
npm run build         # Build server + frontend (tsup + tsc + vite)
npm run build:server  # Build server modules only (tsup → dist-server/)
npm test              # Run all tests (vitest run)
npm run test:watch    # Tests in watch mode
npm run typecheck     # Type check frontend + server
npm run serve:build   # Build + start production server
npm run setup:force   # Re-run interactive setup wizard
npm run setup:auto    # Non-interactive setup (CI-friendly)
npm run db:generate   # Generate Drizzle migrations
npm run db:push       # Push schema to database
```

Run a single test file: `npx vitest run src/__tests__/app.test.tsx`

## Architecture

**Frontend:** React 18 + TypeScript 5.6 + Vite 5. Single-page app with pixel-art map, animated sprites, room navigation, agent CRUD forms, task assignment, and activity feed.

**CLI:** `bin/office.mjs` - globally installable entry point. Resolves `process.cwd()` as project root, creates `.agent-office/` for state, auto-runs init on first use, manages PID file.

**Backend:** Node.js server (`server.mjs` for production, Vite plugin in `vite.config.ts` for dev). Shared API layer in `src/server/`. Default storage: SQLite via Drizzle ORM (zero-config). Optional: PostgreSQL via `DATABASE_URL` env var. API key auth via `OFFICE_API_KEY`. Rate limiting (100 writes/min, 300 reads/min). CSP headers.

**Runtime:** Multi-LLM provider support in `src/runtime/providers/` - Claude Code (default), OpenAI, Ollama. Event-driven task dispatch via `task.queued` events.

**Telegram Bot:** Optional grammy-based bot activated by `TELEGRAM_BOT_TOKEN`. Full agent management via inline keyboards and conversation flows.

**Data flow:** `OfficeProvider` (React context) polls `GET /api/office/snapshot` every 2-3 seconds. Mutations go through provider methods → POST/PATCH/DELETE endpoints → DB or state file → next poll picks up changes.

**Event system:** `src/server/events.ts` - EventEmitter-based pub/sub. API routes emit events, adapters (webhook, Telegram, Slack, GitHub, Linear, Discord, Notion) subscribe.

### Key Files

| File | Role |
|------|------|
| `src/App.tsx` | Main UI - map canvas, room overlays, agent sprites, sidebar panels, forms |
| `src/office-provider.tsx` | React context - polling, state management, CRUD wrappers, validation |
| `src/data.ts` | Seed data - agents, rooms, seats, workday policy |
| `src/world.ts` | Sprite definitions, animation metadata, character sets |
| `src/office-state.ts` | TypeScript type definitions |
| `src/server/api-routes.ts` | Shared API route handlers (pure functions) |
| `src/server/validation.ts` | Shared validation constants and helpers |
| `src/server/events.ts` | Event bus for office events |
| `src/server/json-context.ts` | JSON file-backed ApiContext adapter |
| `src/server/webhook-dispatcher.ts` | Webhook delivery with HMAC + exponential backoff |
| `src/server/paths.ts` | Path resolution for CLI + server |
| `src/server/integrations/` | Slack, GitHub, Linear, Discord, Notion adapters |
| `src/runtime/providers/` | Multi-LLM providers (claude-code, openai, ollama) |
| `src/runtime/types.ts` | Shared runtime provider interface |
| `src/components/shared/CommandPalette.tsx` | Cmd+K command palette |
| `src/db/schema.ts` | Drizzle ORM schema (SQLite) |
| `src/db/schema-pg.ts` | Drizzle ORM schema (PostgreSQL) |
| `src/db/index.ts` | DB connection factory |
| `src/db/migrate.ts` | Auto-migration runner |
| `src/db/seed.ts` | Seed data for empty databases |
| `src/bot/index.ts` | Telegram bot init, start/stop |
| `src/bot/commands.ts` | Bot command handlers |
| `src/bot/keyboards.ts` | Inline keyboard builders |
| `src/bot/callbacks.ts` | Button press handlers |
| `src/bot/conversations.ts` | Multi-step flows (create agent, assign task) |
| `src/bot/notifications.ts` | Push notifications to Telegram |
| `bin/office.mjs` | CLI entry point (global install) |
| `vite.config.ts` | Vite config + dev API plugin |
| `server.mjs` | Production HTTP server (auth, rate limit, CSP) |
| `Dockerfile` | Multi-stage Docker build (uses CLI entry) |
| `docker-compose.yml` | Full stack compose |

### API Endpoints

Shared route handlers in `src/server/api-routes.ts`, used by both dev and prod:

- `GET /api/health` - health check (status, version, DB type, uptime)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Pixel-Process-UG/agent-office](https://github.com/Pixel-Process-UG/agent-office) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
