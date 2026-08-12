---
trigger: always_on
description: This file provides guidance to Claude Code & Codex when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code & Codex when working with code in this repository.

## Code verification

After generating or modifying code, verify it by calling `run_advanced_code_analysis` (if that tool is available in your environment) before committing. Always run `bun run typecheck` and `bun run test` regardless.

## What This Is

Campfire — a collaborative web platform for AI coding agents (Claude Code, Codex, Goose, Aider, OpenHands, OpenClaw, OpenCode). It provides a browser-based interface for running multiple agent sessions with streaming, tool call visibility, and permission control.

Claude Code runs over a long-lived **stdio stream-json transport** by default (`claude -p --input-format stream-json --output-format stream-json --permission-prompt-tool stdio`), wrapped in `claude-stdio-adapter.ts`. The legacy reverse-engineered `--sdk-url` WebSocket transport is still available behind `CAMPFIRE_CLAUDE_TRANSPORT=sdk-url`. All other backends run through stdio adapters (JSON-RPC or stdout parsing).

## Development Commands

```bash
# Dev server (Hono backend on :4567 + Vite HMR on :4567)
cd web && bun install && bun run dev

# Or from repo root
make dev

# Type checking
cd web && bun run typecheck

# Production build + serve
cd web && bun run build && bun run start

# macOS desktop app (Electron shell, Apple Silicon DMG)
make dmg                # stage backend + package desktop/dist/Campfire-<version>-arm64.dmg
cd desktop && bun test  # desktop helper tests (no Electron needed)

# Landing page (campfire.sh) — idempotent: starts if down, no-op if up
# IMPORTANT: Always use this script to run the landing page. Never cd into landing/ and run bun/vite manually.
./scripts/landing-start.sh          # start
./scripts/landing-start.sh --stop   # stop
```

## Testing

```bash
# Run tests
cd web && bun run test

# Watch mode
cd web && bun run test:watch
```

- All new backend (`web/server/`) and frontend (`web/src/`) code **must** include tests when possible.
- Tests use Vitest. Server tests live alongside source files (e.g. `routes.test.ts` next to `routes.ts`).
- A husky pre-commit hook runs typecheck and tests automatically before each commit.
- **Never remove or delete existing tests.** If a test is failing, fix the code or the test. If you believe a test should be removed, you must first explain to the user why and get explicit approval before removing it.
- When creating test, make sure to document what the test is validating, and any important context or edge cases in comments within the test code.

## Component Playground

All UI components used in the message/chat flow **must** be represented in the Playground page (`web/src/components/Playground.tsx`, accessible at `#/playground`). When adding or modifying a message-related component (e.g. `MessageBubble`, `ToolBlock`, `PermissionBanner`, `Composer`, streaming indicators, tool groups, subagent groups), update the Playground to include a mock of the new or changed state.

## Architecture

### Data Flow

```
Browser (React) ←→ WebSocket ←→ Hono Server (Bun) ←→ AgentAdapter (stdio) ←→ Agent CLI
     :4567          /ws/browser/:id       :4567      NDJSON / JSON-RPC     (claude, codex, goose, …)
```

1. Browser sends a "create session" REST call to the server
2. Server spawns the backend CLI as a subprocess (Claude Code: `claude -p --input-format stream-json --output-format stream-json --permission-prompt-tool stdio`)
3. An `AgentAdapter` translates the backend's stdio protocol into normalized browser messages
4. Server bridges messages between the adapter and browser WebSockets (`ws-bridge.ts`)
5. Tool calls arrive as `control_request` (subtype `can_use_tool`) — browser renders approval UI, server relays `control_response` back
6. Legacy path: with `CAMPFIRE_CLAUDE_TRANSPORT=sdk-url`, Claude instead connects back over `/ws/cli/:id` (`--sdk-url` WebSocket, NDJSON)

### All code lives under `web/`

- **`web/server/`** — Hono + Bun backend (runs on port 4567)
  - `index.ts` — Server bootstrap, Bun.serve with dual WebSocket upgrade (CLI vs browser)
  - `ws-bridge.ts` — Core message router. Maintains per-session state (CLI socket, browser sockets, message history, pending permissions). Parses NDJSON from CLI, translates to typed JSON for browsers.
  - `cli-launcher.ts` — Spawns/kills/relaunches Claude Code CLI processes. Handles `--resume` for session recovery. Persists session state across server restarts.
  - `session-store.ts` — JSON file persistence to `~/.campfire/sessions/`. Debounced writes.
  - `session-types.ts` — All TypeScript types for CLI messages (NDJSON), browser messages, session state, permissions.
  - `routes.ts` — backwards-compat shim; the REST API lives in `routes/*.ts` (session CRUD, filesystem browsing, environments, git, cron, gallery, webhooks, agents, races, orchestrator, …).
  - `env-manager.ts` — CRUD for environment profiles stored in `~/.campfire/envs/`.

- **`web/src/`** — React 19 frontend
  - `store.ts` — Zustand store. All state keyed by session ID (messages, streaming text, permissions, tasks, connection status).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stretchcloud/campfire](https://github.com/stretchcloud/campfire) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
