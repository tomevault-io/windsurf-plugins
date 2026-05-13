---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Package Manager

This project uses **pnpm** exclusively. Always use `pnpm` / `pnpm dlx` instead of `npm` / `npx`.

## Project Overview

Lumo is a local-first desktop application for monitoring Claude Code usage. It collects telemetry data via OpenTelemetry Protocol (OTLP) and hook events, then displays usage analytics in a native desktop UI.

**Key characteristics:**
- No accounts, no cloud services, no data leaves your machine
- All data stored locally in SQLite at `~/.lumo/`
- Daemon auto-installed and managed by the desktop app on macOS

### Components

1. **Daemon** (`crates/daemon/`): Standalone HTTP service (Axum) that receives OTLP metrics/logs and hook notifications from Claude Code
2. **Shared Library** (`crates/shared/`): Common database layer (entities, repositories, migrations) used by both daemon and Tauri app
3. **Tauri App** (`src-tauri/`): Desktop application shell with native OS integration, IPC commands, and business services
4. **Frontend** (`packages/ui/`): React-based UI (pnpm workspace package `@lumo/ui`)

## Module Documentation

Each module has its own CLAUDE.md with detailed patterns and guidelines:

- [`crates/daemon/CLAUDE.md`](crates/daemon/CLAUDE.md) - Daemon service architecture
- [`crates/shared/CLAUDE.md`](crates/shared/CLAUDE.md) - Database layer and entity patterns
- [`src-tauri/CLAUDE.md`](src-tauri/CLAUDE.md) - Tauri backend, services, and IPC commands
- [`packages/ui/CLAUDE.md`](packages/ui/CLAUDE.md) - Frontend architecture and component patterns

## Architecture

### System Overview

```
Claude Code
  |
  |-- OTLP logs/metrics (HTTP JSON) --> Lumo Daemon (port 4318) --> SQLite (~/.lumo/lumo.db)
  |-- Hook events (/notify endpoint) ----^                              |
                                                                        v
                                                              Tauri App (reads DB)
                                                                        |
                                                                        v
                                                              React UI (Next.js SSG)
```

### Data Flow

1. **Claude Code** emits OTLP telemetry and hook events
2. **Lumo Daemon** receives them via HTTP endpoints and persists to SQLite
3. **Tauri App** reads from the same SQLite database and serves data to the frontend via IPC
4. **React UI** displays dashboards, session history, tool analytics, and usage insights

### On App Startup

The Tauri app automatically:
1. Initializes the SQLite database and runs migrations
2. Ensures the daemon is installed and running (macOS: `launchd` agent)
3. Configures `~/.claude/settings.json` for OTLP export and hooks
4. Starts background session file watcher and notification poller

### Rust Crates

1. **Daemon** (`crates/daemon/`): OTLP telemetry receiver
   - Standalone HTTP service using Axum
   - Endpoints: `/health`, `/v1/metrics`, `/v1/logs`, `/notify`
   - Persists data to SQLite via shared library
   - Runs independently of Tauri app

2. **Shared Library** (`crates/shared/`): Common database layer
   - Entity definitions with Row/Domain type split
   - Repository pattern for data access (events, metrics, sessions, notifications)
   - Migrations and connection management
   - Used by both daemon and Tauri app

3. **Tauri Backend** (`src-tauri/`): Desktop application shell
   - **Commands** (`commands/`): IPC handlers for frontend communication
   - **Services** (`services/`): Business logic (analytics, trends, tools, usage, wrapped, etc.)
   - **Types** (`types/`): Response types with `#[typeshare]` for TypeScript generation
   - **Daemon** (`daemon/`): Daemon lifecycle management (install, health check, launchd plist)
   - Plugins: log, clipboard, dialog, updater, process, notification, window-state

### Frontend Package

**UI** (`packages/ui/`): Next.js 16 with App Router (SSG mode)
- Pages: Overview, Sessions, Session Detail, Tools, Analytics, Usage, Wrapped
- TanStack Query + Tauri IPC for data fetching
- Tailwind CSS v4 + shadcn/ui components
- ECharts for charts and visualizations

### Full-Stack Data Pipeline

```
Migration (SQL) --> Entity (Row + Domain) --> Repository --> Service --> Command --> Bridge --> useService hook --> UI
```

1. **Migrations** (`crates/shared/migrations/`): SQL schema versioning
2. **Entities** (`crates/shared/src/database/entities/`): `*Row` (FromRow) + Domain (Serialize) + `New*` (insertion)
3. **Repositories** (`crates/shared/src/database/repositories/`): Static methods with `&SqlitePool`
4. **Services** (`src-tauri/src/services/`): Business logic, aggregation, calculations
5. **Commands** (`src-tauri/src/commands/`): `#[command]` IPC handlers
6. **Types** (`src-tauri/src/types/`): Response types with `#[typeshare]` annotation
7. **Bridges** (`packages/ui/src/bridges/`): TypeScript classes wrapping `invoke()` calls
8. **Hooks** (`packages/ui/src/modules/*/use-service.ts`): TanStack Query integration
9. **UI** (`packages/ui/src/modules/*/index.tsx`): Pure rendering components

---

## Prerequisites

- **Node.js** >= 24.12
- **pnpm** >= 10.26

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zhnd/lumo](https://github.com/zhnd/lumo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
