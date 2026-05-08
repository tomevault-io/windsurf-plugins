---
trigger: always_on
description: > This file is read automatically by Claude Code. It contains everything needed to understand, build, test, and contribute to TITAN.
---

# CLAUDE.md — TITAN Project Guide

> This file is read automatically by Claude Code. It contains everything needed to understand, build, test, and contribute to TITAN.

## What is TITAN?

**TITAN (The Intelligent Task Automation Network)** is a premium, autonomous AI agent framework built in TypeScript. It's published as `titan-agent` on npm with 25,000+ installs. Created by Tony Elliott.

- **Current version**: v4.11.1 (semantic versioning — NOT 2026.10.XX)
- **License**: MIT
- **Repo**: https://github.com/Djtony707/TITAN
- **Runtime**: Node.js >= 22, pure ESM

## Quick Reference

| Stat | Value |
|------|-------|
| Providers | 36 (4 native + 32 OpenAI-compatible) |
| Skills | 143 loaded |
| Tools | 248 across 143 loaded skills |
| Channels | 16 (Discord, Telegram, Slack, WhatsApp, Matrix, IRC, Line, Zulip, etc.) |
| Soma | Homeostatic drive layer (v4.0+, opt-in via `organism.enabled`) |
| Tests | 4,655+ across 154+ files (vitest) |
| Default model | `anthropic/claude-sonnet-4-20250514` |
| Gateway port | 48420 |

## Project Structure

```
src/
├── agent/        # Agent core, reflection, sub-agents, orchestrator, goals, initiative, Command Post
├── browsing/     # Shared browser pool (Playwright), CAPTCHA solver (CapSolver)
├── channels/     # 16 channel adapters
├── config/       # Zod-validated config schema
├── context/      # ContextEngine plugin system
├── gateway/      # Express HTTP/WS server + Mission Control v2 (React SPA)
├── mcp/          # MCP Server (JSON-RPC 2.0, stdio + HTTP)
├── memory/       # Memory, learning, graph, relationship, briefings
├── mesh/         # P2P mesh networking (mDNS, WebSocket, HMAC)
├── organism/     # TITAN-Soma: homeostatic drives, pressure loop, hormonal broadcasts, shadow rehearsal (v4.0+)
├── providers/    # LLM provider router + 36 providers
├── skills/       # Builtin skills (143 loaded, 248 tools) + dev + NVIDIA skills
├── utils/        # Constants, helpers, hardware detection
├── voice/        # LiveKit WebRTC voice integration
└── vram/         # VRAM orchestrator (GPU memory management, model swap, leases)
ui/               # React 19 SPA (Vite + Tailwind CSS 4 + React Router v7)
tests/            # 154 vitest test files
e2e/              # Playwright E2E tests (135+ tests, 7 spec files)
```

## Build & Run

```bash
# Install
npm install

# Development (runs from source via tsx)
npm run dev              # CLI
npm run dev:gateway      # Gateway + Mission Control UI

# Build
npm run build            # TypeScript → dist/ (tsup)
npm run build:ui         # React SPA → ui/dist/ (Vite)

# Production
npm start                # Runs from dist/

# Lint & Typecheck
npm run lint
npm run typecheck
```

## Testing

```bash
npm test                 # Run all 4,655 tests
npm run test:watch       # Watch mode
npx vitest run tests/core.test.ts  # Run specific file
```

- Framework: **vitest**
- Tests use heavy `vi.mock()` patterns — see `tests/gateway-extended.test.ts` for the full mock setup
- Mission Control tests: `tests/mission-control.test.ts` (35 tests)

## Version Bumping

When bumping the version, update ALL of these files:
1. `package.json` → `"version"`
2. `src/utils/constants.ts` → `TITAN_VERSION`
3. `tests/core.test.ts` → version assertion
4. `tests/mission-control.test.ts` → version references (4 occurrences)
5. `CHANGELOG.md` → new entry

## Key Architecture Decisions

- **Pure ESM** — No CommonJS. Use `import.meta.url` not `__dirname`.
- **Zod schemas** for all config validation (`src/config/schema.ts`)
- **Provider/model format**: `"provider/model-name"` (e.g., `"anthropic/claude-sonnet-4-20250514"`)
- **Tool execution**: Multi-round loop (up to 25 rounds in autonomous mode)
- **Auth**: Default `auth.mode='token'`. When no `auth.token` is configured, auth is bypassed (open access).
- **Gateway API**: All endpoints under `/api/*`. Auth middleware skips when no token configured.
- **React SPA**: Served from `ui/dist/` at `/`. Legacy dashboard at `/legacy`.

## API Endpoints

Main chat endpoint:
```
POST /api/message
Body: { content, sessionId?, model? }
Returns: { content, sessionId, toolsUsed, durationMs, model }
SSE streaming: Add header `Accept: text/event-stream`
```

Key endpoints:
- `GET /api/health` — Health check
- `GET /api/config` — Full config (nested: `agent.model`, `gateway.auth`, etc.)
- `GET /api/models` — Returns `{ provider: ["provider/model", ...] }` (object, NOT array)
- `POST /api/config` — Update config
- `POST /api/model/switch` — Switch active model
- `GET /api/stats` — System stats
- `GET /api/voice/health` — Voice subsystem status
- `GET /api/goals` — List all goals
- `GET /api/cron` — List cron jobs
- `POST /api/autopilot/toggle` — Enable/disable autopilot
- `POST /api/recipes/:id/run` — Execute a saved recipe
- `POST /api/browser/form-fill` — Direct form fill (bypasses LLM, supports `postClicks`)
- `POST /api/browser/solve-captcha` — Solve CAPTCHA on a given URL via CapSolver
- `GET /api/vram` — GPU VRAM snapshot (state, models, leases)
- `POST /api/vram/acquire` — Request VRAM (auto-swaps models)
- `POST /api/vram/release` — Release a VRAM lease
- `GET /api/vram/check?mb=N` — Dry-run VRAM availability check

### Agent Wakeup API

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Djtony707/TITAN](https://github.com/Djtony707/TITAN) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
