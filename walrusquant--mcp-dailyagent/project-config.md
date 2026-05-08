---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

**Self-hosted data layer for the user's OpenClaw agent.**

The user already tracks tasks/habits/journal/etc. via OpenClaw skills using markdown templates + scripts. This project replaces that fragile setup with a proper Postgres DB behind a typed MCP interface.

- **MCP server** (this repo) — Postgres + `/api/mcp` typed read/write tools + prompt templates. Stores data. Exposes it. Knows nothing about AI.
- **OpenClaw agent** — lives on the user's Mac, runs whatever model the user has configured (**model-agnostic — do NOT assume Claude**), has its own scheduler, bridges to Telegram/WhatsApp/etc. Calls MCP tools to read/write. Generates briefings/reviews/insights and saves them back via MCP write tools.
- **Dashboard** (this repo) — Next.js UI. Viewer + manual editor for the DB. No AI. No generate buttons. If you want AI, you talk to OpenClaw.

Two front doors (OpenClaw agent + dashboard), one DB. Dashboard access is gated by Tailscale (no app-level auth). MCP endpoint is protected by a single bearer token (`MCP_API_KEY`).

## Commands

```bash
npm run dev         # Next.js dev server (Turbopack)
npm run build       # Production build
npm start           # Start production server
npm run lint        # ESLint
npm test            # Vitest once
npm run test:watch  # Vitest watch
npm run db:generate # Generate Drizzle migration from schema diff
npm run db:migrate  # Apply migrations
npm run db:push     # Push schema directly (dev only)
npm run db:studio   # Drizzle studio UI
```

Test files live alongside source in `__tests__/` directories.

## Architecture

### Route Structure (Next.js App Router)

- `src/app/(protected)/` — Dashboard UI: `dashboard`, `tasks`, `habits`, `journal`, `workouts`, `focus`, `goals`, `calendar`, `review`, `spaces`, `settings`. No login — Tailscale gates access.
- `src/app/api/` — API endpoints:
  - `mcp/` — **MCP server endpoint** (Streamable HTTP, stateless, Bearer auth)
  - `tasks/`, `habits/`, `journal/`, `workouts/`, `focus/`, `goals/`, `spaces/`, `tags/`, `calendar/`, `dashboard/` — CRUD for the dashboard
  - `briefing/`, `insights/`, `weekly-review/` — **GET-only**; reads what OpenClaw saved
  - `profile/` — single user's profile
  - `wipe-data/` — Danger Zone nuke button

### MCP Server

Located at `/api/mcp`. Uses the official `@modelcontextprotocol/sdk` with Streamable HTTP transport. Stateless — each request auth'd independently.

**Auth:** Bearer token must match `MCP_API_KEY`. User ID is `SELF_HOSTED_USER_ID` env var. Every request gets full scopes (`all` → expanded via `src/lib/oauth-scopes.ts`).

**Layout** (`src/lib/mcp/`):
- `server.ts` — MCP server factory; registers tools, resources, prompts
- `auth.ts` — Request authentication
- `types.ts` — Shared types (`McpContext`, `QueryResult`)
- `db.ts` — Re-exports Drizzle instance
- `tools/` — Read + write tools (tasks, habits, journal, workouts, focus, goals, spaces, reviews, briefings, **insights**, calendar)
- `resources/` — Read-only resource URIs (`dailyagent://...`)
- `prompts/` — Versioned prompt templates (daily planning, morning briefing, weekly review, habit analysis, productivity report, weekly trends, journal prompt, workout suggestion, goal planning, space planning, etc.). **OpenClaw calls these** — they replace the markdown templates that used to live in OpenClaw skills.
- `queries/` — Shared DB query helpers
- `tools/helpers.ts` — `getAuth`, `checkScope`, `textResult`, `errorResult`, `NOT_AUTHENTICATED`

**How cron works:** OpenClaw has its own scheduler. It fires a task like "do the morning briefing" at 7am, which calls the `morning_briefing` MCP prompt → OpenClaw's LLM reads the returned template + data → generates text → calls `save_daily_briefing` MCP tool to persist → delivers to Telegram. The MCP server itself doesn't run cron, doesn't call an LLM.

### Data Flow

1. **OpenClaw → MCP.** Reads/writes productivity data via `/api/mcp` with `Authorization: Bearer <MCP_API_KEY>`.
2. **Dashboard → API routes → Drizzle → Postgres.** CRUD for all data types. No AI calls.
3. **Dashboard reads what OpenClaw saves.** `daily_briefings`, `weekly_reviews`, `insight_cache` are written by OpenClaw via MCP save tools, displayed read-only in dashboard widgets.
4. **Same DB, two interfaces.** Last-write-wins on shared tables.

### Key Files

- `src/lib/db/client.ts` — Drizzle + postgres.js client (lazy-init so build doesn't need DATABASE_URL)
- `src/lib/db/schema.ts` — All table defs
- `src/lib/db/optimistic.ts` — Optimistic concurrency helper (version-based update)
- `src/lib/auth.ts` — `getUserId()` reading `SELF_HOSTED_USER_ID`
- `src/lib/dates.ts` — Date utilities
- `src/lib/theme.tsx` — ThemeProvider (light/dark/system)
- `src/lib/retry.ts` — Retry utility
- `src/lib/token-validation.ts` — MCP bearer-token validation
- `src/lib/oauth-scopes.ts` — Scope list + `all` expansion
- `src/types/database.ts` — Serialized JSON shapes for dashboard API routes (snake_case); keep in sync with `schema.ts`

### Components

- `src/components/layout/Sidebar.tsx` — Collapsible sidebar with nav + theme toggle (no logout, no admin link)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WalrusQuant/mcp-dailyagent](https://github.com/WalrusQuant/mcp-dailyagent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
