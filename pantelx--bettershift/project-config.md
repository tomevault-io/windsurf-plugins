---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

BetterShift is a self-hosted shift-planning app: Next.js 16 (App Router) + React 19, SQLite via Drizzle, better-auth, next-intl, Tailwind v4 with shadcn/ui primitives. It builds to `output: "standalone"` and ships as a Docker image.

## Commands

```bash
npm run dev            # Dev server
npm run build          # Production build — this is also the type check used in CI
npx tsc --noEmit       # Type check alone, much faster than a build
npm run lint           # ESLint (next core-web-vitals + typescript + @tanstack/query)
npm run i18n           # Translation check (see i18n below)
npm test               # lint + build + i18n — the gate before committing
npm run test:ci        # adds db:generate + db:migrate, mirrors .github/workflows/pr-checks.yml

npm run db:generate    # Create a migration after editing lib/db/schema.ts
npm run db:migrate     # Apply migrations
npm run db:studio      # Drizzle Studio
```

There is no unit-test framework here — "tests" means the lint/build/i18n pipeline. When something fails, run the individual script rather than all of `npm test`.

`npm run release:patch|minor|major` bumps the version and pushes the tag; the release workflow builds the image from it.

## Workflow

Commits follow Conventional Commits (`type(scope): summary`, e.g. `feat(ui):`, `fix(auth):`, `chore:`, `perf:`; `!` before the colon for breaking changes). `scripts/changelog.sh`, called from `.github/workflows/release.yml`, builds the release changelog straight from `git log --pretty=%s --no-merges` between tags — grouped by that prefix, with `refactor|ci|style|test|build` dropped as internal-only.

## Architecture

### Request flow

```text
proxy.ts  →  app/api/**/route.ts  →  getSessionUser(headers)  →  permission check  →  db  →  NextResponse.json()
component →  hooks/use*.ts (TanStack Query) → fetch("/api/…") → cache keyed via lib/query-keys.ts
```

`proxy.ts` is the middleware — Next.js 16 renamed `middleware.ts` to `proxy.ts` and the export is `proxy`. It runs on almost every path (see `config.matcher`) and executes four stages in order:

1. **DB health check**, cached in-module for 10s (5s while unhealthy) with a 2s timeout. Unhealthy redirects everything to `/system-unavailable`; only `/api/health`, `/api/version`, `/api/releases` and `manifest.json` are exempt.
2. **`/share/token/[token]`** — rate limits, validates the token, writes the grant into a cookie, records usage and audit-logs it, then redirects to `/?id=<calendarId>`.
3. **Auth guard** — presence check of the better-auth session cookie only; actual session validation happens in the route handlers. Without a cookie it falls through to guest access or redirects to `/login?returnUrl=…`.
4. **Security headers + CSP** on the response.

Anything added here affects every request, so weigh cost and check the exempt list.

`instrumentation.ts` runs once per server start (Node runtime only): it preloads the version and starts `autoSyncService`. That service is an in-process `setTimeout` scheduler holding a job per external sync — not a cron job, and not shared across replicas.

### Frontend shape

The product is essentially one client page. `app/page.tsx` pulls data hooks (`useCalendars`, `useShifts`, `usePresets`, `useNotes`, `useExternalSync`), pairs them with action hooks (`useShiftActions`, `useNoteActions`) and dialog state (`useDialogStates`), and renders every sheet and dialog through `components/dialog-manager.tsx` — add new dialogs there rather than inline. Real routes exist only for `/login`, `/register`, `/profile`, `/admin/*` and `/system-unavailable`.

View preferences (shifts per day, sorting, note visibility, day highlighting) come from `hooks/useViewSettings.ts` and exist on two levels. The personal view is stored per account in `userPreferences` via `/api/user/view-settings`; guests and `AUTH_ENABLED=false` keep it in `localStorage`, and a signed-in account without a stored view gets the device's values once. A calendar can pin its own view in `calendars.viewSettings` (`null` = off), which replaces the personal view as a whole for everyone with access; the stamp-bar toggle always stays personal, and compare mode always uses the personal view. `lib/view-settings.ts` holds the types, defaults and the sanitiser shared by routes and client.

### Calendar access model

Two independent questions, easy to conflate:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [panteLx/BetterShift](https://github.com/panteLx/BetterShift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
