---
trigger: always_on
description: Extends `~/.hermes/AGENT.md` (base instructions). This file adds project-specific context for working on the Control Hub web application.
---

# Control Hub — Agent Development Guide

§

Extends `~/.hermes/AGENT.md` (base instructions). This file adds project-specific context for working on the Control Hub web application.

§

> **Always read `~/.hermes/AGENT.md` first.** It contains the universal rules, execution loop, and repository structure that apply to all agents.

§

> **For architecture, design rules, and current state, load the `control-hub` skill.** It has the full project documentation.

§

## Development Environment

§

```bash

cd ~/control-hub

npm run dev     # Start dev server (port 3000)

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

│   │   │   ├── agent/files/        # Behaviour file CRUD

│   │   │   ├── agent/agents-md/    # AGENTS.md scanning + CRUD

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

│   │   ├── agent/

│   │   │   ├── agents/            # Agents page (profile CRUD)

│   │   │   └── tools/              # Tools Manager

│   │   ├── page.tsx                # Dashboard

│   │   ├── missions/page.tsx       # Missions page

│   │   ├── cron/page.tsx           # Cron manager

│   │   ├── sessions/page.tsx       # Session browser

│   │   ├── memory/             # Memory CRUD

│   │   ├── config/             # Config editor

│   │   ├── recroom/            # Rec Room — creative activities

│   │   │   ├── page.tsx        # Hub — activity selection

│   │   │   ├── creative-canvas/  # p5.js generative art

│   │   │   ├── ascii-studio/     # ASCII art & animation

│   │   │   └── story-weaver/     # Interactive fiction

│   │   └── layout.tsx              # Root layout with sidebar

│   ├── components/

│   │   ├── recroom/            # Rec Room shared components

│   │   │   ├── PromptBuilder.tsx   # Universal prompt input

│   │   │   ├── OutputViewer.tsx    # Output renderer

│   │   │   ├── ActivityCard.tsx    # Activity preview card

│   │   │   ├── ActivityLayout.tsx  # Page wrapper

│   │   │   └── SaveLoadManager.tsx # Save/load/export

│   │   ├── ui/                     # Primitives (Button, Card, Modal, etc.)

│   │   └── layout/                 # Sidebar, PageHeader

│   ├── lib/

│   │   ├── api.ts                  # Typed fetch wrappers

│   │   ├── config-schema.ts        # Config section definitions

│   │   ├── theme.ts                # Shared theme maps

│   │   ├── utils.ts                # timeAgo, timeUntil, formatBytes

│   │   └── recroom/

│   │       └── prompt-templates.ts # LLM system prompts per activity

│   └── types/

│       └── hermes.ts               # All TypeScript interfaces

├── data/

│   ├── missions/                   # Mission JSON files

│   └── templates/                  # Custom template JSON files

├── public/                         # Static assets

├── next.config.ts                  # Next.js config

├── tailwind.config.ts              # Tailwind config

└── package.json

```

§

## Key Conventions

§

- **TypeScript strict** — no `any`, no `@ts-ignore`

- **API routes return `{ data?, error? }`** — all routes use `ApiResponse<T>` from `@/types/hermes`

- **Error logging** — all catch blocks call `logApiError(route, context, error)` from `@/lib/api-logger`

- **Loading + error states** for every async operation

- **Destructive actions need confirmation**

- **NEVER write to `~/.hermes/` directly** — always through API endpoints

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

- `src/lib/hermes.ts` — `PATHS`, `CH_DATA_DIR`, `HERMES_HOME`, `getDefaultModelConfig()`, `getDiscordHomeChannel()`

§

## Git Workflow

§

**Always work on `dev` branch. Never commit to `main`.**

§

```bash

# Before starting work

cd ~/control-hub

git checkout dev

git pull origin dev

§

# After making changes

git add -A

git commit -m "type: description"

git push origin dev

§

# Create PR for review

curl -X POST https://api.github.com/repos/Daniel-Parke/hermes-control-hub/pulls \


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Daniel-Parke/hermes-control-hub](https://github.com/Daniel-Parke/hermes-control-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
