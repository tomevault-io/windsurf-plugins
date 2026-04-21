---
trigger: always_on
description: Personal AI agent with multi-channel messaging, browser automation, and persistent memory.
---

# dorabot

Personal AI agent with multi-channel messaging, browser automation, and persistent memory.

## Workflow Principles

When asked to plan or design, STOP and plan from the user's perspective BEFORE writing or analyzing code. Do not start coding autonomously when the user wants a plan, task list, or discussion first.

## Debugging & Investigation

When asked to investigate or debug something, run tests and scripts FIRST to observe actual behavior before reading/analyzing code extensively. Prefer empirical testing over code tracing.

For SDK/API questions, check official SDK documentation and API docs via Codex-agent-sdk skill FIRST, not the local codebase. Only grep local code after understanding the external API surface.

## Architecture

- **Backend**: Node.js + TypeScript, Codex Agent SDK (`@anthropic-ai/Codex-agent-sdk`)
- **Gateway**: WebSocket RPC server over Unix socket `~/.dorabot/gateway.sock` (`src/gateway/server.ts`)
- **Desktop**: Electron + Vite + React in `desktop/`
- **Channels**: WhatsApp (Baileys), Telegram (grammy)
- **Database**: SQLite (`~/.dorabot/dorabot.db`) via better-sqlite3, WAL mode
- **Tools**: MCP server — `message`, `browser`, `screenshot`, `calendar` (4 tools), `goals` (4 tools)
- **Browser**: Playwright-core via CDP, persistent profile at `~/.dorabot/browser/profile/`, port 19222
- **Skills**: Markdown files in `./skills/` and `~/.dorabot/skills/`, YAML frontmatter for metadata
- **Providers**: Pluggable — Codex (Agent SDK, default) or OpenAI Codex (Codex SDK)
- **Calendar**: RFC 5545 iCal RRULE-based scheduling (replaced cron)

## Build

```bash
# backend
npm run build                        # tsc → dist/

# desktop (all-in-one via electron-vite)
cd desktop && npm run build          # → desktop/out/{main,preload,renderer}

# dev
npm run dev                          # gateway with auto-reload (tsx --watch)
npm run dev:desktop                  # desktop with HMR (electron-vite dev)
npm run dev:cli                      # interactive CLI mode
```

## Key Files

### Core
- `src/agent.ts` — `runAgent()` and `streamAgent()`, orchestrates SDK `query()` with config, skills, workspace, hooks
- `src/db.ts` — SQLite database singleton (`getDb()`), schema creation, WAL mode, foreign keys
- `src/system-prompt.ts` — dynamic system prompt builder
- `src/config.ts` — config loading, merging, path allowlisting via `isPathAllowed()`
- `src/workspace.ts` — loads SOUL.md, USER.md, AGENTS.md, MEMORY.md from `~/.dorabot/workspace/`
- `src/index.ts` — CLI entry point

### Gateway
- `src/gateway/server.ts` — WebSocket RPC server, 71 RPC methods, agent run queue, tool approval system
- `src/gateway/types.ts` — `WsMessage`, `WsResponse`, `WsEvent`, `SessionInfo` types
- `src/gateway/session-registry.ts` — in-memory session tracking, backed by SQLite `sessions` table
- `src/gateway/channel-manager.ts` — start/stop WhatsApp and Telegram monitors

### Sessions & Database
- `src/session/manager.ts` — SQLite-backed session/message storage (sessions + messages tables)
- `scripts/migrate-to-sqlite.ts` — one-time migration from legacy JSONL files to SQLite

### Providers
- `src/providers/index.ts` — provider factory, singleton management
- `src/providers/types.ts` — provider type definitions
- `src/providers/Codex.ts` — Codex provider (Agent SDK, default)
- `src/providers/codex.ts` — OpenAI Codex provider (dynamic import)

### Channels
- `src/channels/types.ts` — `InboundMessage`, `ChannelHandler`, `SendOptions`
- `src/channels/whatsapp/` — Baileys socket, monitor, send/edit/delete, QR login
- `src/channels/telegram/` — grammy bot, long-polling runner, markdown→HTML conversion

### Tools
- `src/tools/index.ts` — MCP server creation, registers 12 custom tools
- `src/tools/messaging.ts` — `messageTool` + channel handler registry pattern
- `src/tools/browser.ts` — single tool with `action` discriminator, 37 actions (input, navigation, snapshots, scripts, console/network inspection)
- `src/tools/calendar.ts` — `schedule`, `list_schedule`, `update_schedule`, `cancel_schedule` (RFC 5545 RRULE)
- `src/tools/goals.ts` — `goals_view`, `goals_add`, `goals_update`, `goals_propose`
- `src/tools/screenshot.ts` — standalone screenshot tool

### Calendar
- `src/calendar/scheduler.ts` — iCal RRULE-based scheduler, auto-migration from legacy cron_jobs

### Browser
- `src/browser/manager.ts` — find Chromium, launch via CDP, singleton page
- `src/browser/refs.ts` — DOM snapshot → uid refs, resolve ref → Playwright Locator
- `src/browser/actions.ts` — 30+ action functions: open, snapshot, click, click_at, drag, type, fill, fill_form, select, press, hover, upload_file, scroll, cookies, evaluate, console/network inspection, pdf, etc.

### Skills
- `src/skills/loader.ts` — load from dirs, eligibility checks, prompt matching
- `skills/` — 9 built-in skills: agent-swarm-orchestation, github, himalaya, image-gen, macos, meme, onboard, polymarket, remotion

### Desktop
- `desktop/src/App.tsx` — root layout, 8-tab navigation, resizable 3-panel layout
- `desktop/src/hooks/useGateway.ts` — central state, WebSocket RPC, 40+ methods, event handling, streaming
- `desktop/src/views/` — Chat, Goals, Channel, Skills, Settings, Soul, Status, Tools

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [suitedaces/dorabot](https://github.com/suitedaces/dorabot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
