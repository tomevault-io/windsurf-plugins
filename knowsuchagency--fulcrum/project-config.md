---
trigger: always_on
description: Fulcrum is the Vibe Engineer's Cockpit. A terminal-first tool for orchestrating AI coding agents across isolated git worktrees.
---

## Project Overview

Fulcrum is the Vibe Engineer's Cockpit. A terminal-first tool for orchestrating AI coding agents across isolated git worktrees.

**Philosophy**:
- Terminal-first AI agent orchestration. Agents (Claude Code, OpenCode, etc.) run in terminals as-is—no abstraction layer, no wrapper APIs.
- Three task types: **Git** (isolated worktrees), **Scratch** (isolated directories, no git), **Manual** (no agent/directory).
- Persistent terminals organized in tabs for work that doesn't fit neatly into task worktrees.
- App deployment via Docker Compose with automatic DNS/tunnel routing.
- System monitoring for Claude instances and resource usage.

## Quick Navigation

| Topic | Section |
|-------|---------|
| Running dev server, building | [Development](#development) |
| Running tests | [Testing](#testing) |
| CLI commands | [CLI](#cli) |
| Tech stack, folder structure | [Architecture](#architecture) |
| Database schema, migrations | [Database](#database) |
| Settings, config files | [Configuration](#configuration) |
| Docker Compose apps | [App Deployment](#app-deployment) |
| Slack/Discord/Pushover | [Notifications](#notifications) |
| WhatsApp/Discord/Telegram/Slack/Email | [Messaging](#messaging) |
| Desktop app packaging | [Desktop App](#desktop-app) |
| dtach, PTY management | [Terminal Architecture](#terminal-architecture) |
| Log format, locations | [Logging](#logging) |
| Directory structure | [File Organization](#file-organization) |

## Where to Find Things

**Adding a new feature:**
- Frontend page → `frontend/routes/` (file-based routing)
- API endpoint → `server/routes/` (Hono handlers)
- Business logic → `server/services/`
- Database table → `server/db/schema.ts`

**Modifying existing features:**
- Tasks/worktrees → `server/routes/tasks.ts`, `server/services/task-service.ts`, `server/services/task-status.ts` (status transitions, recurrence)
- Messaging channels → `server/services/channels/`, `server/routes/messaging.ts`
- App deployment → `server/routes/apps.ts`, `server/services/` (docker/cloudflare/traefik)
- Calendar integration → `server/services/caldav/`, `server/routes/caldav.ts`, `frontend/components/caldav/`
- Google integration → `server/services/google/`, `server/services/google-oauth.ts`, `server/routes/google.ts`, `server/routes/google-oauth.ts`
- Agent memory → `server/routes/memory.ts`, `server/services/memory-service.ts`, `server/routes/memory-file.ts`, `server/services/memory-file-service.ts`
- Unified search → `server/routes/search.ts`, `server/services/search-service.ts`
- Settings → `server/lib/settings/`, `frontend/routes/settings/`
- Secrets (fnox) → `server/lib/settings/fnox.ts`, `cli/src/utils/fnox-setup.ts`

**UI components:**
- Shared components → `frontend/components/ui/` (shadcn)
- Feature components → `frontend/components/{feature}/`

**Testing:**
- Test files live next to source: `*.test.ts`

## Development

All commands are mise tasks. Run `mise tasks` to list available commands.

```bash
mise run dev          # Start frontend and backend dev servers
mise run build        # Build for production
mise run up           # Build and start production server as daemon
mise run down         # Stop the daemon server
mise run check        # Run all checks (lint + typecheck + version)
mise run db:generate  # Generate new migration from schema changes
mise run db:migrate   # Apply pending migrations
mise run db:studio    # Open Drizzle Studio GUI
mise run cli:build    # Build CLI package for npm distribution
mise run bump         # Bump patch version (or: bump major, bump minor)
mise run desktop:package  # Package desktop app for current platform
```

For type checking, just run `mise run build` - it catches type errors and is faster than running separate typecheck commands.

## Testing

Run tests via mise to get filtered output that shows only failures:

```bash
mise run test         # Run all tests (quiet mode, errors only)
mise run test -- -v   # Run all tests with verbose output
mise run test:watch   # Run tests in watch mode
mise run test:file server/routes/config.test.ts  # Run specific test file
```

**Critical**: Never run `bun test` directly. Always use mise tasks for test isolation.

The mise test tasks set `HOME` and `FULCRUM_DIR` to temp directories **before** Bun starts. This is necessary because Bun caches `os.homedir()` at process startup, before any JavaScript runs. Without this isolation, tests that write to settings files would corrupt production `~/.fulcrum/settings.json` and `~/.claude/settings.json`.

## CLI

The `fulcrum` package provides a global CLI:

```bash
fulcrum up             # Start the bundled server as daemon
fulcrum down           # Stop the daemon
fulcrum status         # Check if server is running
fulcrum doctor         # Check all dependencies and versions
fulcrum mcp            # Run as MCP server (stdio transport)
fulcrum tasks          # List/manage tasks
fulcrum notifications  # Manage notification settings
fulcrum notify <title> <message>  # Send notification
```

## Architecture

### Frontend (`frontend/`)
- **React 19** with TanStack Router (file-based routing in `frontend/routes/`)
- **TanStack React Query** for server state

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [knowsuchagency/fulcrum](https://github.com/knowsuchagency/fulcrum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
