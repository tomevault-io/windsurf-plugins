---
trigger: always_on
description: Self-hosted task management PWA. This is the authoritative reference for contributing to this codebase. Canonical filename: `AGENTS.md` (symlinked as `CLAUDE.md` for Claude Code compatibility).
---

# OpenTask

Self-hosted task management PWA. This is the authoritative reference for contributing to this codebase. Canonical filename: `AGENTS.md` (symlinked as `CLAUDE.md` for Claude Code compatibility).

**Start here:** [Critical Requirements](#critical-requirements) (data-loss prevention), [Commands](#commands) (build/test), [Route Handler Guide](#route-handler-guide) (API patterns), [Development and Testing](#development-and-testing) (what to run when). Reference sections (task model, due dates) are at the end — consult them when working on related features.

See also: `README.md` (project overview), `CONTRIBUTING.md` (quickstart), `TASKS.md` (backlog). Environment-specific details (server addresses, deploy scripts, credentials) are in `CLAUDE.local.md` (gitignored).

## Architecture

Next.js 16 (App Router) + React 19 + React Compiler + TypeScript + SQLite (better-sqlite3) + NextAuth 5 + Tailwind CSS 4 + Shadcn UI. Mobile-first PWA optimized for iOS. Basic offline support: `public/sw.js` caches the app shell so navigation works offline, but there is no offline data access or mutation queuing. Uses Next.js standalone output mode for deployment. A native iOS companion app (`ios/`) wraps the PWA in a WKWebView and adds APNs push notifications.

Additional docs: `docs/SPEC.md` (product requirements), `docs/ROADMAP.md` (planned features), `docs/AUTOMATION.md` (external API integration), `DEV_LOG.md` (design decisions and narrative context), `docs/NOTIFICATIONS.md` (push architecture), `docs/IOS-DEV-LOG.md` (iOS history), `docs/openapi.yaml` (REST API schema), `docs/AI.md` (AI architecture and quality testing), `docs/TASK-MODEL.md` (task model reference), `docs/DESIGN.md` (design philosophy).

### Documentation site

The public docs site lives in a separate repo (`opentask-docs`, typically at `~/working_dir/opentask-docs`) (VitePress, deployed to `opentask.mcnitt.io/docs/`). When you change features, API behavior, or configuration in this repo, the docs site may need a corresponding update. Key sync points:

| Change in this repo                          | Update in opentask-docs                           |
| -------------------------------------------- | ------------------------------------------------- |
| API routes or response format                | `openapi.json` (convert from `docs/openapi.yaml`) |
| AI features or provider options              | `setup/ai.md`, `setup/configuration.md`           |
| Environment variables                        | `setup/configuration.md`                          |
| Core behavior (snooze, priority, recurrence) | `concepts/` pages                                 |
| New feature or major change                  | `overview.md`                                     |

See `~/working_dir/opentask-docs/CLAUDE.md` for build/deploy instructions and full sync details.

### Source layout

- `src/core/` — Business logic (no UI): auth, db, errors, projects, tasks, recurrence, undo, validation, notifications, webhooks, review, stats, ai, activity, export
- `src/components/` — React components (see directory for full inventory)
- `src/components/ui/` — Shadcn UI primitives (button, input, checkbox, dialog, sheet, etc.)
- `src/hooks/` — Custom React hooks (`useSelectionMode.ts`, `useGroupSort.ts`, `useTimezone.ts`, `useKeyboardNavigation.ts`, etc.)
- `src/app/api/` — REST API routes with three auth methods (Bearer tokens + proxy headers + session cookies)
- `src/app/` — Pages (App Router): root (`/`), login, tasks/[id], settings, history, archive, trash
- `src/lib/` — Utilities (`api-response.ts`, `format-task.ts`, `format-date.ts`, `format-rrule.ts`, `logger.ts`, `priority.ts`, `toast.ts`, `utils.ts`, etc.)
- Real-time sync via Server-Sent Events (pushes task changes to open browser tabs): `src/app/api/sync/stream/`, `src/lib/sync-events.ts`, `src/hooks/useSyncStream.ts`
- `src/types/` — Domain types (`index.ts`), API route types (`api.ts`), NextAuth augmentation (`next-auth.d.ts`), Web Speech API types (`speech-recognition.d.ts`)
- `src/instrumentation.ts` — Next.js server init hook that starts notification cron jobs
- `ios/` — Native iOS companion app (see [iOS App](#ios-app-ios) section)
- `assets/` — Source logo and branding files (Pixelmator sources + exported PNGs). After updating source files in `assets/`, copy exports to `public/`.

### UI vocabulary

| Term        | Component                                 | Description                                                                 |
| ----------- | ----------------------------------------- | --------------------------------------------------------------------------- |
| Top bar     | `src/components/Header.tsx`               | Fixed header with snooze button, task count, and navigation                 |
| Quick panel | `src/components/QuickActionPanel.tsx`     | Grid of snooze/priority buttons used in task detail and the dashboard modal |
| Action bar  | `src/components/SelectionActionSheet.tsx` | Floating black bar with Done/Details/More buttons shown during selection    |

### Database

- **Singleton**: Access via `getDb()` from `@/core/db`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trentmcnitt/opentask](https://github.com/trentmcnitt/opentask) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
