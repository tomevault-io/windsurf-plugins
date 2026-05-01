---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Prismer.AI is an open-source academic research platform combining paper reading, writing, data analysis, and multi-agent AI workflows. It ships as a self-hosted Docker container with a Next.js frontend and uses OpenClaw for agent orchestration.

**Licensing:** Workspace (`web/`) and Docker (`docker/`) are Apache-2.0.

## Repository Structure

```
web/                     # Next.js 16 frontend (React 19, TypeScript, Tailwind 4)
docker/                  # Container infrastructure & OpenClaw integration
  base/                  # Base Docker image (ubuntu + LaTeX/Jupyter/Prover services)
  plugin/                # OpenClaw plugins (prismer-im, prismer-workspace)
  gateway/               # Container reverse proxy (zero-dependency)
  config/                # Agent config (openclaw.json, SOUL.md, AGENTS.md)
  templates/             # Agent personality templates (mathematician, finance, etc.)
  scripts/               # Python CLI tools (latex, jupyter, component, sync)
docs/                    # Architecture docs, roadmap, i18n translations
skills/                  # OpenClaw academic skills (placeholder)
```

## Build & Development Commands

### Frontend (`web/`)

```bash
npm run dev              # Next.js dev server (requires Node 18+)
npm run build            # prisma generate && next build (standalone output)
npm run start            # Production server
npm run lint             # ESLint
npm run sync:server      # WebSocket agent/sync server (port 3456)
```

### Database (`web/`)

```bash
npm run db:generate      # Generate Prisma Client (output: src/generated/prisma/)
npm run db:push          # Push schema to database (creates SQLite on first run)
npm run db:migrate       # Create migration
npm run db:studio        # Prisma Studio GUI
```

First-time setup requires: `npm run db:generate && npm run db:push`

### Tests (`web/`)

```bash
npm test                 # Vitest unit tests (single run)
npm run test:watch       # Vitest watch mode
npm run test:coverage    # Coverage report
npm run test:e2e         # All Playwright E2E tests
npm run test:layer1      # Playwright: container + plugin + API (real agent)
npm run test:layer2      # Playwright: mock frontend rendering + trace
npm run test:layer3      # Playwright: full E2E, no mocks + trace
npm run test:report      # Show Playwright HTML report
```

### Docker

```bash
# Dev mode (agent container only, frontend runs locally)
docker compose -f docker/docker-compose.dev.yml up

# Lightweight single-container
docker compose -f docker/docker-compose.lite.yml up

# Full OpenClaw setup
docker compose -f docker/docker-compose.openclaw.yml up
```

Container gateway exposes port 16888 locally. Frontend at `web/` connects to it via `CONTAINER_GATEWAY_URL`.

## Architecture

### Tech Stack

- **Next.js 16** with App Router, standalone output, React Compiler enabled
- **React 19**, **Tailwind CSS 4**, **shadcn/ui** (new-york style, config in `web/components.json`)
- **Zustand 5** with composition pattern (10 stores composed into one)
- **Prisma 6** with SQLite (dev) / MySQL (prod); schema at `web/prisma/schema.prisma`
- **TypeScript 5** strict mode; path alias `@/*` → `./src/*`

### Workspace Store Composition (`web/src/app/workspace/stores/`)

The workspace is a composition of 10 Zustand stores, not a single monolithic store:

- `workspaceStore.ts` — Composition root, workspace metadata
- `chatStore.ts` — Messages & conversation
- `taskStore.ts` — Task management (state machine)
- `timelineStore.ts` — Event timeline
- `layoutStore.ts` — UI layout (panel sizes, collapsed state)
- `componentStore.ts` — Editor component states
- `agentInstanceStore.ts` — Agent instance state
- `demoStore.ts` — Demo mode
- `syncActions.ts` — Sync orchestration (80+ actions bridging frontend ↔ agent)

Global stores (`web/src/store/`): `flowStore.ts`, `skillStore.ts`

### Sync Layer (`web/src/lib/sync/`)

Rules-based sync engine that bridges frontend components with the external OpenClaw agent:

- `SyncMatrixEngine.ts` — Rules-based sync controller
- `defaultMatrix.ts` — Sync rules per data type (which fields sync, direction, conflict resolution)
- `componentStateConfig.ts` — Field-level sync configuration
- `useAgentConnection.ts` — WebSocket connection lifecycle hook
- `useContentSync.ts` — Content CRUD with auto-sync hook

### Directive Protocol (`web/src/lib/directive/`)

Agent → Frontend communication flow:
```
Agent tool → Python CLI → JSON file → Bridge API → Directive Queue → Frontend
```
Directive types: `switch_component`, `update_content`, `compile_complete`, `jupyter_cell`, etc.

### Container Integration (`web/src/lib/container/`)

- `openclawGatewayClient.ts` — Gateway WebSocket client
- `staticAgentConfig.ts` — Static agent configuration
- `orchestrator.ts` / `dockerOrchestrator.ts` — Container lifecycle management

### API Routes (`web/src/app/api/`)

- `v2/im/bridge/[workspaceId]/` — Current standard: IM-Container bridge (forwards messages to agent, receives directives)
- `workspace/[id]/` — Workspace CRUD + messages/tasks/timeline/files/notes/component-states

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Prismer-AI/Prismer](https://github.com/Prismer-AI/Prismer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
