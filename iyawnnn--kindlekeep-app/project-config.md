---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

`kindlekeep-app` is the **frontend only** (React 19 + Vite dashboard) half of KindleKeep, a zero-cost uptime/security monitoring product. The backend (.NET 10 API + SignalR hubs + Postgres) lives in a separate repo, `kindlekeep-api`, per the polyrepo strategy in `ARCHITECTURE.md` (section 18) — do not expect backend code here, and don't try to "complete" the stack by adding a server into this repo.

`ARCHITECTURE.md` in the repo root is a large product/architecture manifest (vision, market analysis, full target data schema, roadmap). Treat it as aspirational context, not a description of current code — most of the backend schema and features it describes do not exist in this repo. The parts of it that do reflect real conventions here (naming, design system, package manager rationale) are folded into this file below.

## Commands

Package manager is **pnpm** (see `ARCHITECTURE.md` §5 — chosen for disk footprint / lockfile parity with Vercel). Don't use npm/yarn.

```bash
pnpm dev       # start Vite dev server
pnpm build     # tsc -b (project references, type-check only) && vite build
pnpm lint      # eslint .
pnpm preview   # preview a production build locally
```

There is no test runner configured in this repo (no test script, no test files) — don't assume Vitest/Jest exist.

## Environment

Local dev reads `.env.local` (gitignored):
- `VITE_API_URL` — backend API base URL (defaults to `http://localhost:5247` in code if unset, see `src/lib/axios.ts` and `src/features/monitors/hooks/useSignalR.ts`)
- `VITE_SIGNALR_HUB_URL` — SignalR hub URL

The backend must be running (from the sibling `kindlekeep-api` repo) for auth, monitor data, and the SignalR pulse stream to work — the UI alone will not do much against a stub.

## Architecture

### Auth
Auth is OAuth-based against the backend; the frontend just stores the resulting JWT in `localStorage` under `jwt_token` (`src/App.tsx`). `ProtectedRoute` in `App.tsx` gates dashboard routes by checking for that token's presence (no expiry/validation client-side — that's the backend's job). `/auth-callback` (and the legacy `/auth/callback/:provider`) reads a `token` query param, stores it, and redirects to `/dashboard`.

### Data flow
- **REST**: `src/lib/axios.ts` exports a shared `api` axios instance with an interceptor that attaches `Authorization: Bearer <jwt_token>` from `localStorage` to every request. Use this instance for all HTTP calls, not a bare `axios` import.
- **Real-time**: `src/features/monitors/hooks/useSignalR.ts` wraps `@microsoft/signalr`, connecting to `${VITE_API_URL}/hubs/pulse` with the JWT as the access token. On `ReceivePulse` it patches `useMonitorStore` directly (status/latency), so components generally don't need to handle socket events themselves — just read from the store. It supports an optional `monitorId` to subscribe/unsubscribe to a single monitor's stream (used by the debug terminal / monitor detail view) and an `onLog` callback for raw log streaming (`ReceiveLogStream`), used by the xterm-based debug terminal.
- **Client cache**: TanStack Query is wired up in `main.tsx` (`refetchOnWindowFocus: false`, `retry: 1`) for one-shot fetches; SignalR is the source of truth for live status changes, Query/axios for everything else.
- **Client state**: Zustand (`useMonitorStore` in `src/features/monitors/store/`) holds the monitor list. Mutations (`toggleMonitor`, `deleteMonitor`) apply optimistic updates and roll back on request failure — follow that pattern for new store mutations rather than waiting on the server round-trip.

### Structure
Feature-based, not type-based, under `src/features/<feature>/{components,hooks,store,types}`. Cross-feature/shared UI goes in `src/components/ui/`. Route-level components live flat in `src/pages/`. There are no path aliases configured (`tsconfig.app.json` has none) — imports are relative.

### Routing
Single `react-router-dom` tree in `App.tsx`. Authenticated pages are wrapped individually in `<ProtectedRoute><Layout>...</Layout></ProtectedRoute>`; `Layout` renders the shared header/nav. Landing/login/signup are public and unwrapped.

## Design system


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iyawnnn/kindlekeep-app](https://github.com/iyawnnn/kindlekeep-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
