---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

RaceIQ is a full-stack racing telemetry analysis app for Forza Motorsport 2023, F1 25, Assetto Corsa Competizione, Assetto Corsa Evo, and iRacing. UDP and native Windows telemetry sources feed a Bun server, SQLite storage, and a React dashboard. See [architecture overview](docs/architecture/overview.md).

## Commands

```bash
# Development (starts both server and client)
bun run dev

# Server only (Bun with --watch, port 3117)
bun run dev:server

# Client only (Vite with portless)
bun run dev:client

# Tests (Bun test runner)
bun run test                        # use bun run test, not bun test (sets --timeout 60000)
bun test --timeout 60000 test/parser.test.ts   # single test file

# Database
bun run db:push       # sync Drizzle schema to SQLite (dev introspection only — see note below)
bun run db:generate   # generate Drizzle migration files (not used at runtime — see note below)

# Production build (client bundle + compiled server binary → dist/)
bun run build

# Run production build
bun run start

# Build Windows installer
bun run build:installer

# Client-specific
cd client && bun run build   # production build (tsc + vite)
cd client && bun run lint    # ESLint

# Dump mode (develop without a running game — captures raw packets)
bun run dev:dump:fm            # dump Forza Motorsport packets
bun run dev:dump:f1            # dump F1 2025 packets
bun run dev:dump:acc           # dump ACC packets

# AI development (Mastra agent playground)
bun run mastra:studio          # Studio UI (localhost:3000) reading the running dev server's in-process Mastra API (:3117)

# Utility scripts
bun run extract:tracks         # extract track data from game files
bun run laps:export            # export lap data
bun run laps:import            # import lap data
bun run lighthouse             # run Lighthouse audit on local dev server
```

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `SERVER_PORT` | `3117` | HTTP/WebSocket server port |
| `UDP_PORT` | `5301` | Game telemetry UDP listen port |
| `DATA_DIR` | `./data` | Database and settings directory |

### Development Onboarding Flag

Pass `--onboarding false` to the full development command to skip the Setup
Wizard and open any RaceIQ page directly:

```bash
bun run dev --onboarding false
```

Use `--onboarding true` to force the Setup Wizard, or omit the flag to use the
persisted onboarding state normally. The server-side override is development
only and does not change `settings.json`; production builds ignore the flag.

## Architecture

### Three-layer monorepo: `server/`, `client/`, `shared/`

**Server (Bun + Hono)**
- `server/index.ts` — Thin executable entry; `server/runtime/boot.ts` owns ordered startup
- `server/runtime/udp-listener.ts` — UDP socket listening for game telemetry packets
- `server/games/` — Game-owned parsers/adapters plus generic packet dispatch — see [Adding a New Game](#adding-a-new-game)
- `server/routes/index.ts` — Hono app composition; bounded route groups live in `server/routes/`
- `server/runtime/websocket-manager.ts` — WebSocket manager, 30Hz throttled broadcast to all connected clients
- `server/telemetry/live-pipeline.ts` — Telemetry processing pipeline (normalize → suspension fill → lap detect → sector track → pit track → track calibration → broadcast)
- `server/lap-detection/detector.ts` — Detects lap boundaries from telemetry stream (per-game factory via adapter)
- `server/live-strategy/` — Live sector timing and pit/fuel/tire estimates
- `server/lap-analysis/corners.ts` — Game-aware racing-corner identification
- `server/ai/` — AI analysis system (see [AI Analysis System](#ai-analysis-system))
- `server/db/schema.ts` — Drizzle ORM schema (profiles, sessions, laps, corners, lapAnalyses, compareAnalyses, trackOutlines)
- `server/db/*-queries.ts` — Responsibility-scoped database query modules
- `server/db/migrations.ts` — Hand-rolled migration list (SQL array, version-tracked)
- `server/db/index.ts` — Runs migrations on startup via custom runner
- `server/runtime/platform/tray.ts` — System tray integration (Windows)
- `server/runtime/update/check.ts` — Auto-update checker

### Database migration approach

Drizzle is used **only as a query builder and type-safe schema reference** — NOT for runtime migrations. Schema changes are managed via a hand-rolled migration system in `server/db/migrations.ts`. The app compiles to a self-contained Windows binary (`raceiq.exe`); Drizzle's `migrate()` reads SQL files from disk at runtime, which would break single-binary distribution. The custom system embeds all migration SQL directly in the compiled binary.

**To add a schema change:**
1. Edit `server/db/schema.ts` (keeps Drizzle types in sync)
2. Add a new entry at the bottom of `server/db/migrations.ts` with the next version number and the raw SQL
3. Do NOT use `bun run db:push` to apply schema changes — it is for dev introspection only and must never drop `schema_migrations` (protected via `tablesFilter` in `drizzle.config.ts`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SpeedHQ/RaceIQ](https://github.com/SpeedHQ/RaceIQ) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
