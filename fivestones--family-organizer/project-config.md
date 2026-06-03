---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Rules
- Do your very best to write code that is both perfectly functional and exetremely good looking.
- When you finish coding something, do not test the UI unless you have been explicitly instructed to do so. I will do the UI testing for you. That means you must skip preview verification (unless I ask you to do so for a specific case).

## Commands

```bash
# Development
npm run dev              # Start Next.js with Turbopack

# Mobile (Expo)
npm run mobile:start     # Start Expo dev server
npm run mobile:ios       # Run on iOS simulator

# Testing
npm test                 # Run all Vitest tests
npm run test:watch       # Vitest watch mode
npm run test:coverage    # Vitest with coverage report
npm run test:e2e         # Playwright E2E tests (auto-starts dev server on port 3000)
npm run test:all         # Vitest + Playwright

# Run a single test file
npx vitest run test/unit/lib/task-scheduler-date-logic.node.test.ts

# Run live InstantDB permissions smoke test (requires env vars)
npm run test:perms:live

# InstantDB schema management (local dev)
INSTANT_CLI_DEV=1 INSTANT_CLI_API_URI="http://localhost:8888" npx instant-cli@latest push
INSTANT_CLI_DEV=1 INSTANT_CLI_API_URI="http://localhost:8888" npx instant-cli@latest pull
```

## Architecture

This is a **Next.js 16 + InstantDB** family management app. InstantDB is the sole database—there is no traditional backend DB. All data sync happens client-side via the InstantDB React SDK, with the server-side Admin SDK used only for privileged operations (auth token minting, admin writes).

### Key architectural layers

**Client DB** (`lib/db.ts`): `@instantdb/react` initialized with `instant.schema.ts` for typed queries. All client components use `db.useQuery(...)` for live data and `db.transact(...)` for writes.

**Server/Admin** (`lib/instant-admin.ts`): `@instantdb/admin` for operations requiring elevated privileges. Used in API routes under `app/api/`.

**Schema** (`instant.schema.ts`): Single source of truth for all entities and their links. Modify here and push with `instant-cli`. Key entities: `familyMembers`, `chores`, `choreAssignments`, `choreCompletions`, `taskSeries`, `tasks`, `allowanceEnvelopes`, `allowanceTransactions`, `calendarItems`, `deviceSessions`.

**Permissions** (`instant.perms.ts`): CEL-based rules pushed to InstantDB. Changes must be pushed to take effect.

### Auth / device access model

Two-layer auth:
1. **Device auth** (`lib/device-auth.ts`): A `DEVICE_ACCESS_KEY` cookie gates the entire app. `middleware.ts` enforces this — unauthenticated devices are redirected to `/activate`. The `/api/device-activate` endpoint validates a shared secret from the environment and sets the cookie.
2. **Principal/role** (`lib/instant-principal-types.ts`, `lib/parent-mode.ts`): Within the app, family members log in by PIN. Principals are `kid` | `parent` | `unknown`. Parent elevation goes through `lib/parent-elevation-rate-limit.ts` and mints short-lived tokens via `/api/instant-auth-parent-token`. `components/auth/ParentGate.tsx` is the component-level guard for parent-only pages and `components/auth/LoginModal.tsx` handles PIN entry and family member selection.

### Web app pages and features

**Routes (App Router):**
- First, you have to activate the app. If you try to load any route before doing this, you will get a 404 not found error. After the server is running, get `DEVICE_ACCESS_KEY` from the .env file, and go to `http://<ip>:<port>/?activate=<DEVICE_ACCESS_KEY>`. This will activate you and immediately redirect you to `/`. Then you will be able to get to any of the other routes without a 404 error.
- `/activate` — Device activation screen (enter shared `DEVICE_ACCESS_KEY`). You should not use this to activate; it's for human use and you can use the method described above for quicker access.
- `/` — Main chore dashboard (`ChoresTracker` + `ChoreList`). Family member sidebar, date carousel, add/edit/delete chores, task series checklists with file attachment previews, marks completions.
- `/calendar` — Multi-week calendar with drag-and-drop events. Supports both Gregorian and Nepali (Bikram Samvat) views. Displays chore dates and calendar items.
- `/task-series` — Task series manager (parent-only). Lists all series with status filters (draft/pending/in_progress/archived), batch delete, progress tracking.
- `/task-series/new` and `/task-series/[seriesId]` — TipTap-based rich text editor for creating/editing task series. Supports slash commands, nested task indentation, drag-and-drop reordering, and day-break markers.
- `/familyMemberDetail` — Per-member allowance management: multiple currency envelopes, deposits/withdrawals/transfers between envelopes or to other members, transaction history, savings goals, recurring allowance schedule setup.
- `/allowance-distribution` — Parent-only allowance distribution workflow. Selects a period, calculates weight-based completion percentage, handles fixed up-for-grabs rewards, previews and executes payouts.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fivestones/family-organizer](https://github.com/fivestones/family-organizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
