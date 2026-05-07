---
trigger: always_on
description: This guide enables AI coding agents to work productively in the GSD Task Manager codebase. Follow these project-specific conventions and workflows for best results.
---

# Copilot Instructions for GSD Task Manager

This guide enables AI coding agents to work productively in the GSD Task Manager codebase. Follow these project-specific conventions and workflows for best results.

## Architecture Overview

### Core Stack
- **Next.js 16 App Router**: All routes in `app/`. Matrix view: `app/(matrix)/page.tsx`. Dashboard: `app/(dashboard)/dashboard/page.tsx`. PWA install: `app/(pwa)/install/page.tsx`.
- **Client-side only**: No server rendering. All components use `"use client"`.
- **Data Layer**: IndexedDB via Dexie v4 (`lib/db.ts`, current schema v13). CRUD in `lib/tasks.ts`. Live queries via `useTasks()` hook (`lib/use-tasks.ts`).
- **Quadrant System**: Tasks classified by `urgent`/`important` booleans → 4 quadrants. Logic in `lib/quadrants.ts`.
- **Schema Validation**: Zod schemas in `lib/schema.ts` for all data types.

### Database Schema (v13)
- **Tables**: `tasks`, `archivedTasks`, `smartViews`, `notificationSettings`, `syncQueue`, `syncMetadata`, `deviceInfo`, `archiveSettings`, `syncHistory`, `appPreferences`
- **Task Fields**: Core (id, title, description, urgent, important, quadrant, completed, completedAt, dueDate, createdAt, updatedAt) + Advanced (recurrence, tags, subtasks, dependencies, notifyBefore, snoozedUntil, estimatedMinutes, timeSpent, timeEntries)
- **Indexes**: Performance-critical indexes on `quadrant`, `completed`, `dueDate`, `completedAt`, `createdAt`, `updatedAt`, `*tags`, `*dependencies`, `notificationSent`
- **Migrations**: Schema changes always require migration in `lib/db.ts`. See `DATABASE_ARCHITECTURE.md` for full ERD.

### Key Architectural Decisions
1. **PocketBase cloud sync**: Optional cloud sync via self-hosted PocketBase server. Tasks stored as plaintext (user owns the server). OAuth authentication with Google/GitHub.
2. **Last-write-wins conflict resolution**: Conflicts resolved using `client_updated_at` timestamps. BFS algorithm prevents circular dependencies in task graph.
3. **Modular components**: Large files split into <300 line modules. Example: `lib/sync/` has 20+ focused modules vs monolithic sync engine.
4. **Pure analytics functions**: All metric calculations in `lib/analytics/` are side-effect-free for testability and composability.
5. **Transaction-based batch operations**: `lib/bulk-operations.ts` ensures atomicity (all-or-nothing) for multi-task updates.

## Component Patterns

### UI Components
- **Primitives** (`components/ui/`): shadcn-style components (button, dialog, input, etc.). No size prop on buttons—use variant: "primary" | "subtle" | "ghost".
- **Domain Components**: `matrix-board/` (modular orchestration), `matrix-column.tsx`, `task-card.tsx`, `task-form/` (modular form with validation hook).
- **Dialogs**: All use Radix UI Dialog primitive. Example: `import-dialog.tsx`, `bulk-tag-dialog.tsx`.
- **Hooks**: Custom hooks like `useTasks()`, `useViewTransition()`, `useSmartViewShortcuts()` encapsulate complex logic.

### Smart Views & Command Palette
- **Smart Views**: Saved filter configurations (`smartViews` table). Pin up to 5 in header with keyboard shortcuts (1-9, 0=clear).
- **Command Palette**: Global ⌘K/Ctrl+K shortcut (`command-palette.tsx`). Includes quick actions, navigation, and search.
- **Settings**: Single consolidated dialog (`components/settings/settings-dialog.tsx`) with iOS-style grouped sections (Appearance, Notifications, Cloud Sync, Archive, Data & Storage, About). Opened directly from the header gear icon — no intermediate "quick" panel.

### Sync Architecture
- **Frontend**: `lib/sync/` with modular architecture: `pb-sync-engine.ts` (push/pull), `pb-realtime.ts` (SSE subscriptions), `pb-auth.ts` (OAuth), `pocketbase-client.ts` (SDK singleton), `task-mapper.ts` (field mapping), `sync-coordinator.ts` (orchestrator).
- **Backend**: Self-hosted PocketBase server at `https://api.vinny.io` (AWS EC2). OAuth with Google/GitHub. API rules enforce per-user data isolation.
- **Realtime**: PocketBase SSE (Server-Sent Events) for instant cross-device updates with echo filtering via `device_id`.

### MCP Server Integration
- **Purpose**: Enable Claude Desktop to access/analyze tasks via natural language.
- **Location**: `packages/mcp-server/` (standalone npm package, Node.js 18+).
- **20 Tools**: Read (7), Write (5), Analytics (5), System (3). All write operations support `dryRun` mode.
- **Config**: `~/Library/Application Support/Claude/claude_desktop_config.json` with `GSD_POCKETBASE_URL`, `GSD_AUTH_TOKEN`.

## Developer Workflows

### Essential Commands
- `bun install` — Install deps (generates `bun.lock`)
- `bun dev` — Dev server at http://localhost:3000
- `bun typecheck` — TypeScript type checking (no emit)
- `bun lint` — ESLint with Next.js config
- `bun run test` — Vitest CI mode (`bun test` invokes bun's built-in runner, not vitest)
- `bun run test:watch` — Vitest watch mode
- `bun run test -- --coverage` — Coverage report (target: ≥80% statements/lines/functions, ≥75% branches)
- `bun run build` — Production build (includes typecheck)
- `bun run export` — Static export for S3/CloudFront

### CloudFront Deployment

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vscarpenter/gsd-task-manager](https://github.com/vscarpenter/gsd-task-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
