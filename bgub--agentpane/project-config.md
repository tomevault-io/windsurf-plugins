---
trigger: always_on
description: Web UI for AI coding agents. `npx agentpane` starts the API server (port 3456) and a Next.js frontend (port 6767). No separate deployment — everything runs locally.
---

# AgentPane

Web UI for AI coding agents. `npx agentpane` starts the API server (port 3456) and a Next.js frontend (port 6767). No separate deployment — everything runs locally.

Communication with agents uses ACP (Agent Client Protocol) — JSON-RPC 2.0 over stdio.

## Quick Start

```sh
npx agentpane          # starts API (:3456) + frontend (:6767)
```

For development: `npm run dev` starts both Next.js dev server (port 6767) and Hono API server (port 3456) via Turbo. Next.js rewrites `/api` requests to localhost:3456.

## Tech Stack

- **Monorepo:** Turborepo + npm workspaces (`apps/*`)
- **Backend (`apps/server`):** Hono + Effect.ts services/layers, Node.js server on port 3456. Published to npm as `agentpane`.
- **Frontend (`apps/web`):** Next.js 16 (App Router, standalone output), React 19, Tailwind CSS 4, TanStack React Query.
- **Docs (`apps/docs`):** Astro + Starlight.
- **Database:** SQLite via `@effect/sql-sqlite-node` + `better-sqlite3` at `~/.agentpane/agentpane.db`
- **ACP:** `@agentclientprotocol/sdk` + `@zed-industries/claude-agent-acp` + `@zed-industries/codex-acp`

## Architecture

### Two-Process Production Server

`bin/agentpane.js` starts two processes: the Hono API server on :3456 (imported directly) and a forked Next.js standalone server on :6767. In development, the Next.js dev server on :6767 rewrites `/api/*` to :3456. No auth or CORS — everything is same-origin.

### Backend (`apps/server/`)

Hono HTTP server with Effect.ts service layers composed into `ManagedRuntime`:

- `src/index.ts` — Entry point, Hono app + `serve()` on port 3456
- `src/lib/schema.ts` — Effect Schema classes for `Session`, `Turn`, `MessageBlock`, and error types
- `src/lib/db.ts` — `SqliteLive` layer (SQLite connection, migrations, WAL mode)
- `src/lib/runtime.ts` — `AppRuntime = ManagedRuntime.make(AppLayer)`
- `src/lib/session-repo.ts` — `SessionRepo` service (CRUD for sessions, turns, message blocks, settings)
- `src/lib/acp-client.ts` — `AcpClient` facade composing ConnectionManager, EventHub, PromptEngine, WriteQueue
- `src/lib/connection-manager.ts` — Agent subprocess lifecycle, ACP connection setup, session load/resume
- `src/lib/prompt-engine.ts` — Prompt execution orchestration, turn persistence, token tracking
- `src/lib/event-hub.ts` — Session-level `EventBroadcaster` management with idle cleanup
- `src/lib/event-broadcaster.ts` — Ring buffer SSE implementation (replay on reconnect via `Last-Event-ID`)
- `src/lib/write-queue.ts` — Batched DB writes (50ms debounce per session, crash recovery)
- `src/lib/write-ops.ts` — Write operation types for the queue
- `src/lib/acp-client-callbacks.ts` — Agent→client callbacks (sessionUpdate, file I/O, terminal management)
- `src/lib/acp-types.ts` — ACP protocol interfaces
- `src/lib/providers.ts` — Agent provider config + binary resolution (walks up `node_modules/.bin/` for npm hoisting compatibility)
- `src/routes/sessions.ts` — All Hono route handlers
- `src/routes/validation.ts` — Request validation helpers

### Layer Composition

```
AppLayer =
  AcpClient.layer
  → PromptEngine.layer
  → ConnectionManager.layer
  → EventHub.layer
  → WriteQueue.layer
  → SessionRepo.layer
  → SqliteLive
```

### API Routes (all in `apps/server/src/routes/sessions.ts`)

```
GET    /api/health                        → health check
GET    /api/metrics                       → uptime, memory, ACP stats
GET    /api/git-branch?cwd=...            → current git branch
GET    /api/settings/:key                 → get setting
PUT    /api/settings/:key                 → save setting

GET    /api/sessions                      → list sessions
POST   /api/sessions                      → create (optionally auto-connect)
GET    /api/sessions/status               → { connected: id[], prompting: id[] }
GET    /api/sessions/:id                  → get session
PATCH  /api/sessions/:id                  → rename session
DELETE /api/sessions/:id                  → disconnect + delete

GET    /api/sessions/:id/conversation     → turns with message blocks
GET    /api/sessions/:id/token-usage      → aggregated token counts
POST   /api/sessions/:id/prompt           → send prompt (text, images, resource links)
POST   /api/sessions/:id/cancel           → cancel in-progress prompt
POST   /api/sessions/:id/permission       → respond to permission request

GET    /api/sessions/:id/commands         → available slash commands
GET    /api/sessions/:id/config           → config options
POST   /api/sessions/:id/config           → set config option
GET    /api/sessions/:id/mode             → current mode
POST   /api/sessions/:id/mode             → set mode
GET    /api/sessions/:id/agent-sessions   → agent's session history

POST   /api/sessions/:id/connect          → connect agent
DELETE /api/sessions/:id/connect          → disconnect agent
GET    /api/sessions/:id/events           → SSE stream (reconnect-safe via Last-Event-ID)
```

### Frontend (`apps/web/`)

Next.js 16 App Router — pure UI, no backend dependencies. Same-origin with the API via rewrites.

- `src/app/layout.tsx` — Root layout with providers (theme, React Query)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bgub/agentpane](https://github.com/bgub/agentpane) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
