---
trigger: always_on
description: Guidance for Claude Code (claude.ai/code) when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) when working in this repository.

## Project Overview

Custom smart display system (a Dakboard / MagicMirror replacement). Web-based, runs on a Raspberry Pi in Chromium kiosk mode, portrait 1080x1920 by default. All data is local JSON under `data/`. No database, no cloud. Solo pre-release project: no backwards-compatibility shims or migration paths unless asked.

## Commands

```bash
npm run dev          # Dev server (Next.js)
npm run build        # Production build
npm run lint         # ESLint
npm test             # Unit tests (vitest)
npx vitest run src/lib/__tests__/config.test.ts   # One test file
npm run test:e2e     # Playwright E2E (needs `npm run build` first)
npx playwright test --project=meta     # Coverage ratchets; run first when adding a module
npx playwright test --project=editor   # One surface's E2E specs
npm run test:shell   # Bash tests for scripts/*.sh (run after editing any shell script)
```

Preflight gate before any commit: `npx tsc --noEmit`, `npm run lint`, `npm test` all pass.

## Tech Stack

Next.js 16 + React 19 (App Router), Tailwind v4, Zustand (editor state), @dnd-kit (editor drag-and-drop), Framer Motion (editor panels only; screen transitions use the View Transitions API in `ScreenRotator`), Vitest, Playwright. Path alias `@/*` maps to `./src/*`.

## Architecture

### Route groups (`src/app/`)
| Group | Path | Purpose |
|---|---|---|
| `(display)` | `/display`, `/display/[displayId]` | Fullscreen kiosk view. When the displays registry exists, `/display` renders the main display inline rather than redirecting (Chromium `--app` mode duplicates its window on a 307). |
| `(editor)` | `/editor` | Layout editor plus Settings. |
| `(remote)` | `/remote` | Phone remote and family surfaces (chores, meals, timers, lists, photos). Manages data; the editor styles the display. |
| `(auth)` | `/login` | Authentication. |
| `api/` | `/api/*` | One `route.ts` per endpoint. All external services (weather, calendar, stocks...) are proxied server-side to hold secrets and avoid CORS. |

### Code shared between surfaces
The same domain is often edited from two or three places: the editor, `/remote`, and sometimes the wall. What they share and what they must not is settled:

- **Rules go in `src/lib/<domain>-*.ts`** as pure functions: what a valid edit is, what an action does to the data, what an absent value means. `meal-settings.ts` and `meal-plan-actions.ts` are the worked examples, and `chore-form-presentation.ts` is the older one. A rule written twice is the shape that lets a fix land on one surface and not the other.
- **Components both surfaces render go in `src/components/<domain>/`** (`family/`, `meals/`, `timetable/`), not inside `src/app/(remote)/remote/components/` where the editor cannot reach them.
- **Markup usually should not be shared.** The phone is touch-sized and inline-styled against the `remote` dictionary; the editor is not a touch surface, uses Tailwind, reads the `editor` dictionary, and its fields carry `data-field-id` for the settings search. One component serving both takes the styling system, density, dictionary and save model as parameters, which costs more than it saves. Share the rules underneath instead.

### API auth tiers
Every route under `src/app/api/` opens with one guard from `src/lib/auth.ts`, and picking the wrong one is the easiest security mistake to make:
- `requireSession`: a logged-in editor or phone user. Rejects display bearer tokens. Use for anything that writes config or family data.
- `requireDisplayAuth`: a session cookie or the kiosk's display bearer token, with an optional trusted-IP bypass. Use for reads a wall display needs and the few writes it may make (tick a to-do, post status).
- `requireAdoptedDisplay`: LAN plus presence in `config.displays`. Used by Pi telemetry.
- `requireSudo` (`src/lib/sudo-grant.ts`): for system actions (upgrade, WiFi, hostname, restart). Answers 409 when the service account has no passwordless sudo, which the editor turns into a password prompt that repairs the grant.
Proxy routes built with `cachedProxyRoute` are the exception to "opens with a guard": they declare `auth: 'display' | 'session'` on the factory config instead, so grepping for a guard clause will not find them.
`src/proxy.ts` sits in front of all of it: auth on/off, the IP allowlist, and rejection of cross-origin writes. Writes are default-deny there, but GET protection is a hand-maintained allowlist (`PROTECTED_GET_ROUTES`), so a meta ratchet requires every route to declare a posture one of those ways or name itself in `PUBLIC_ROUTES` with a reason.

### Config schema migrations
`config.json` carries a schema version. `src/lib/migrations/` holds one `vN-to-vN+1.ts` per step and `migrateUp` runs on every read; the latest version is derived from the list, so adding a file is the whole bump. Any change to the shape of `ScreenConfiguration`, `Screen`, `ModuleInstance` or a module config needs a migration, not a read-time shim. Plugin config shapes migrate through `src/lib/plugin-config-migration.ts`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [home-screens/home-screens](https://github.com/home-screens/home-screens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
