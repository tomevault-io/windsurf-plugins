---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test Commands

### Commands to run servers

Never run these commands yourself or kill related processes. The user is responsible for running the server. If you need to run or restart one of these commands, ask the user to do it for you.

- web (client+api): `pnpm dev` (port 3000)
- web client dev only: `pnpm dev:web-client` (port 5173 in dev)
- web api dev only: `pnpm dev:web-api` (port 3000, passes through web-client)
- MCP server dev: `pnpm dev:mcp-server`
- Telegram bot dev: `pnpm dev:telegram-bot`

### Root Level

Use these commands for investigating, testing and linting.
DO NOT cd into packages. you MUST stay in root and run commands like `pnpm test|build` from the repo root.
CRITICAL: Run `pnpm check:commit` to verify code before requesting a commit.

- IMPORTANT: Use `pnpm logs:tail` to investigate unified logs produced by `pnpm dev`
- Build all packages: `pnpm build` (always run from the repo root, not from individual package dirs)
- Build for production: `pnpm build:production`
- Lint: `pnpm lint`
- Format check: `pnpm lint:format`
- Commit check (required before commit): `pnpm check:commit`
- Format fix: `pnpm format`
- Unit tests (default): `pnpm test`
- Unit tests only: `pnpm test:unit`
- MCP server integration tests: `pnpm test:integration:mcp-server` (uses vitest with global setup)
- Agent loop integration tests: `pnpm test:integration:agent-loop` (uses VCR caching, see below)
- Agent loop tests (record): `pnpm test:integration:agent-loop:record` (re-record cassettes)
- Agent loop tests (playback): `pnpm test:integration:agent-loop:playback` (CI mode, no API calls)
- E2E tests: `pnpm test:e2e`
- Full test suite: `pnpm test:all`
- CI test suite: `pnpm test:ci` (excludes telegram-bot integration tests)
- CI test suite with all tests: `pnpm test:ci:all` (requires ANTHROPIC_API_KEY)
- Run single test: `pnpm vitest:run src/path/to/file.test.ts`
- TypeScript check: `pnpm tsc:check`
- MCP server test: `pnpm test:mcp-server` (this lets you run commands against mcp-server)
- Check unused dependencies: `pnpm knip`

### Package-Specific

- Build specific package: `pnpm build:client|api|core|mcp-server|telegram-bot`

### Backup & Restore

- Interactive backup: `pnpm backup:interactive` (prompts for CouchDB URL)
- Interactive restore: `pnpm restore:interactive` (prompts for CouchDB URL)
- With URL flag: `pnpm backup:interactive -- --url http://admin:password@localhost:5984`
- Verify backup: `pnpm backup:verify`
- Automated backup scheduler: `pnpm backup:auto` (runs continuous backup cycle)
- Retention policy: `pnpm backup:retention` (apply retention policy to cleanup old backups)
- Retention dry-run: `pnpm backup:retention --dry-run` (preview what would be deleted)

**Backup Scheduler Options:**

- `--interval <time>` - Backup interval (e.g., "24h", "1d", "30m") - default: 24h
- `--pattern <glob>` - Database name pattern to backup (e.g., "eddo*\*") - default: eddo*\*
- `--run-once` - Run single backup cycle and exit
- `--no-verify` - Skip backup verification
- `--no-retention` - Skip retention policy

**Environment Variables for Backups:**

- `BACKUP_DIR` - Directory for backup files (default: ./backups)
- `BACKUP_DATABASE_PATTERN` - Glob pattern for databases to backup (default: eddo\_\*)

### CLI & Mock Data

- CLI interface: `pnpm cli`
- Test CLI: `pnpm cli:test`
- Populate mock data: `pnpm populate-mock-data`
- Populate mock data (dry run): `pnpm populate-mock-data:dry-run`

## Architecture Overview

This is a GTD-inspired todo and time tracking application built as a monorepo with multiple packages:

- **web-client**: React/TypeScript frontend with PouchDB for offline-first storage (port 5173 in dev)
- **web-api**: Hono API server for authentication and CouchDB proxy (port 3000)
- **core**: Common types, utilities, and data models across packages
- **mcp-server**: MCP (Model Context Protocol) server for external integrations (port 3002)
- **telegram-bot**: Telegram bot with AI agent capabilities using Anthropic Claude

### Key Architectural Patterns

- **Database-Centric**: PouchDB serves as both storage and state management (no Redux/Zustand)
- **Offline-First**: Local browser storage with real-time sync via PouchDB changes feed
- **Versioned Data Model**: Automatic migration system between schema versions (alpha1 → alpha2 → alpha3)
- **Calendar Week View**: UI organized around calendar weeks with date-range queries
- **GTD-Style Contexts**: Todos grouped by context (e.g., "work", "private") in Kanban-style layout

### Data Flow

1. Components access PouchDB directly via `usePouchDb()` hook
2. Database changes trigger React re-renders through changes feed
3. No centralized state store - PouchDB is the source of truth
4. Design documents provide MapReduce views for efficient querying

### Current Data Model (Alpha3)

```typescript
interface TodoAlpha3 {
  _id: string; // ISO timestamp of creation
  active: Record<string, string | null>; // Time tracking entries
  blockedBy?: string[]; // Todo IDs that must complete first (use with gtd:blocked tag)
  completed: string | null;
  context: string; // GTD context

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [walterra/eddoapp](https://github.com/walterra/eddoapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
