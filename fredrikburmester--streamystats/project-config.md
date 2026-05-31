---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Streamystats is a self-hosted analytics platform for Jellyfin media servers. It provides watch statistics, AI-powered recommendations, security monitoring, and data visualization. Users deploy it via Docker alongside their Jellyfin instance. It continuously syncs data from Jellyfin and presents it through a web dashboard. It's a Bun monorepo with two services: a Next.js web app and a Hono job server, backed by VectorChord PostgreSQL 17 (pgvector + tsvector).

## Commands

```bash
# Development
bun run dev              # Start both services (job-server + next.js)
bun run dev:nextjs       # Next.js only
bun run dev:job-server   # Job server only (with --watch)

# Build & Production
bun run build            # Build database package + Next.js
bun run start            # Start both services

# Database
bun run db:generate      # Generate migrations from schema.ts
bun run db:migrate       # Run pending migrations
bun run db:studio        # Launch Drizzle Studio

# Code Quality
bun run lint:fix         # Biome lint with fixes
bun run format:fix       # Biome format with fixes
bun run typecheck        # TypeScript check (both apps)

# Testing
cd apps/nextjs-app && bun test           # Run all tests
cd apps/nextjs-app && bun test file.ts   # Run single test file
```

## Architecture

```
┌─────────────┐       ┌───────────────┐       ┌─────────────────────┐
│   Browser    │──────▶│  Next.js      │──────▶│  Job Server         │
│              │ :3000 │  (BFF + UI)   │ :3005 │  (Hono + pg-boss)   │
└─────────────┘       │               │       │                     │
                      │  Server       │       │  Session Poller     │
                      │  Components   │       │  Scheduled Jobs     │
                      │  API Routes   │       │  SSE Event Bus      │
                      │  Server       │       └──────────┬──────────┘
                      │  Actions      │                  │
                      └───────┬───────┘                  │
                              │                          │
                              ▼                          ▼
                      ┌──────────────────────────────────────────┐
                      │  VectorChord PostgreSQL 17               │
                      │  pgvector (embeddings) + tsvector (FTS)  │
                      │  :5432                                   │
                      └──────────────────────────────────────────┘
                              │
                              │ Jellyfin API calls
                              ▼
                      ┌──────────────────┐
                      │  Jellyfin Server │
                      │  (external)      │
                      └──────────────────┘
```

```
apps/
├── nextjs-app/          # Web UI + API routes (port 3000)
│   ├── app/             # Next.js App Router
│   │   ├── (app)/       # Main app layout group
│   │   └── api/         # REST endpoints (authenticated)
│   ├── components/      # React components
│   │   └── ui/          # shadcn/ui components
│   ├── lib/
│   │   ├── db/          # Database query functions (25+ files)
│   │   ├── ai/          # AI integration (chat, embeddings)
│   │   ├── auth.ts      # Session authentication
│   │   └── api-auth.ts  # API token validation
│   └── hooks/           # Custom React hooks
│
└── job-server/          # Background job processor (port 3005, Hono)
    └── src/
        ├── jobs/        # Job definitions (scheduler, session-poller, embeddings, geolocation, security-sync, server-jobs)
        ├── events/      # SSE event bus (job-events.ts)
        └── routes/      # HTTP endpoints for job management

packages/
└── database/            # Shared database layer
    └── src/
        ├── schema.ts    # Drizzle ORM schema (single source of truth)
        └── connection.ts
```

**Next.js app** (port 3000): Web UI via App Router with server components, API routes for external integrations, server actions for mutations. Acts as BFF — the browser never talks to the job-server directly.

**Job server** (port 3005): Hono HTTP server with pg-boss for persistent job queues. Runs a session poller (in-memory tracking of active Jellyfin playback) and scheduled cron jobs for data sync. Publishes SSE events for real-time progress.

**PostgreSQL**: VectorChord image (PostgreSQL 17 + pgvector). Extensions: `vector` (embeddings), `uuid-ossp` (ID generation). Full-text search via `tsvector` columns with GIN indexes, updated by database triggers.

### Communication

- Next.js → job-server: HTTP via `JOB_SERVER_URL` (e.g., `http://job-server:3005`)
- Both services → PostgreSQL: via `DATABASE_URL`
- Job-server → Jellyfin: HTTP via stored server URLs (uses `internalUrl` when configured, falls back to `url`)
- Next.js → Jellyfin: HTTP from server components/API routes using `getInternalUrl(server)`
- Browser → Jellyfin: only for login (credentials flow), never for data

### Deployment


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fredrikburmester/streamystats](https://github.com/fredrikburmester/streamystats) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
