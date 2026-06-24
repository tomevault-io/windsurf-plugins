---
trigger: always_on
description: Extends `~/.hermes/AGENTS.md` (base instructions). This file adds project-specific context for working on the Control Hub web application.
---

# Control Hub — Agent Development Guide

§

Extends `~/.hermes/AGENTS.md` (base instructions). This file adds project-specific context for working on the Control Hub web application.

§

> **Always read `~/.hermes/AGENTS.md` first.** It contains the universal rules, execution loop, and repository structure that apply to all agents.

§

> **For architecture, design rules, and current state, load the `control-hub` skill.** It has the full project documentation.

§

## Development Environment

§

```bash

cd ~/control-hub

npm run dev     # Start dev server (PORT from .env.local; scripts/bootstrap/setup.sh defaults 42069–42100)

npm run build   # Production build

npm run start   # Start production server

```

§

## Project Structure

§

```

control-hub/

├── src/

│   ├── app/

│   │   ├── api/                    # REST API routes

│   │   │   ├── agent/files/[key]/    # Behaviour file read/update

│   │   │   ├── agent/profiles/     # Agent profile CRUD

│   │   │   ├── tools/              # Toolset config per platform

│   │   │   ├── missions/           # Mission CRUD + dispatch

│   │   │   ├── config/             # Config YAML CRUD

│   │   │   ├── cron/               # Cron job management

│   │   │   ├── sessions/           # Session browser

│   │   │   ├── memory/             # Holographic memory CRUD

│   │   │   ├── agents/             # Running agent detection

│   │   │   ├── monitor/            # Aggregated system status

│   │   │   ├── templates/          # Custom template CRUD

│   │   │   └── ...                 # Other endpoints

│   │   ├── page.tsx                # Dashboard

│   │   ├── (main)/                 # sessions, memory, logs (route group — no /main URL prefix)

│   │   ├── orchestration/          # cron, missions, chat

│   │   ├── operations/             # agents, skills, tools, personalities

│   │   ├── config/                 # Config editor + models hub

│   │   ├── recroom/story-weaver/   # Story Weaver

│   │   └── layout.tsx              # Root layout with sidebar

│   ├── components/

│   │   ├── layout/                 # Sidebar, PageHeader, AppPageShell

│   │   ├── missions/, cron/, models/, memory/, story-weaver/, ui/

│   ├── lib/

│   │   ├── db.ts, db/migrations/   # SQLite + baseline schema

│   │   ├── hermes-home.ts          # HERMES_HOME resolution

│   │   ├── hermes-agent-runtime.ts # Active Hermes paths + gateway URLs

│   │   ├── *-repository.ts         # missions, cron, models, credentials, …

│   │   ├── sync/                   # Background sync + SyncScheduler

│   │   ├── backends/hermes.ts      # Mission dispatch

│   │   ├── api-fetch.ts            # Shared client fetch helper

│   │   ├── schema/                 # Mission + template Zod schemas

│   │   ├── config-schema.ts        # Config section definitions

│   │   ├── theme.ts, utils.ts, …

│   └── types/

│       └── hermes.ts               # All TypeScript interfaces

├── tests/

│   ├── unit/                       # Jest unit + API tests

│   ├── e2e/                        # Playwright (incl. app-routes nav matrix)

│   ├── integration/                # Docker install/update harness (Python)

│   ├── jest.setup.ts

│   └── __mocks__/better-sqlite3.cjs

├── scripts/                        # bootstrap/, application/ch-deploy.sh, tooling/, lib/, hardware/, …

├── scripts/git-hooks/              # Optional pre-push (see docs/CONTRIBUTING.md)

├── docs/                           # Technical documentation index → docs/README.md

├── next.config.ts                  # Next.js config

├── tailwind.config.ts              # Tailwind config

└── package.json

```

Next.js static files (favicon, `robots.txt`, etc.) go in a `public/` directory at the repo root when you add them—the folder is not committed empty; the production `Dockerfile` runs `mkdir -p public` before build.

§

## Key Conventions

§

- **TypeScript strict** — no `any`, no `@ts-ignore`

- **API routes return `{ data?, error? }`** — all routes use `ApiResponse<T>` from `@/types/hermes`

- **Error logging** — all catch blocks call `logApiError(route, context, error)` from `@/lib/api-logger`

- **Loading + error states** for every async operation

- **Destructive actions need confirmation**

- **Do not bypass the API to edit Hermes or Control Hub state on disk** — use routes so path validation and registry-aware resolution apply

- **`.env` keys displayed as `sk-...abcd` only**

- **Use `js-yaml` for YAML parsing**

- **String concatenation for paths, NOT `path.join`** (Turbopack NFT tracing issue)

- **Build before deploy:** `npm run build` must pass

- **Security** — whitelist body fields in PUT handlers (no mass assignment), validate paths with `path.resolve()` + `startsWith()`

§

## Shared Utilities

§

- `src/lib/utils.ts` — `parseSchedule()`, `CronJobData`, `messageSummary()`, `timeAgo()`, `timeUntil()`, `formatBytes()`

- `src/lib/api-logger.ts` — `logApiError()`, `safeJsonParse()`, `safeReadJsonFile()`

- `src/lib/paths.ts` — `PATHS` (Control Hub–owned dirs), `CH_DATA_DIR`, `getChScriptsDir()`, `getChHardwareLogDir()`, `getDiscordHomeChannel()`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Daniel-Parke/PatterStage](https://github.com/Daniel-Parke/PatterStage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
