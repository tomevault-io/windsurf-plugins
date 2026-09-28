---
trigger: always_on
description: UT Austin campus event discovery app. Expo / React Native client (iOS, Android, web) talking to a
---

# Longhorn Loop

UT Austin campus event discovery app. Expo / React Native client (iOS, Android, web) talking to a
Cloudflare Worker backend on D1.

Ticket ids (`LOOP-###`) appear throughout the code comments and map to the team's tracker. Existing
comments in this repo are unusually detailed and explain _why_ a decision was made — read them before
changing the thing they describe, and keep that standard when adding code.

## Repo layout

```
app/          Expo Router client (screens, components, hooks, client libs)
server/       Cloudflare Worker (Hono routes, scrapers, D1 migrations)
shared/       Dependency-free modules imported by BOTH client and server
site/         Static marketing + privacy pages
docs/         Design notes (org-profiles.md)
```

`shared/` must stay free of React, SVG imports, and server-only APIs — the client imports it via the
`@/` alias, the server via relative paths.

## Commands

Client (repo root):

```
npx expo start          # dev server
npm run lint            # eslint (CI)
npm run format:check    # prettier (CI)
npm run typecheck       # tsc --noEmit (CI)
```

Server (`server/`):

```
npm run dev             # wrangler dev --env local
npm run dev:lan         # same, bound to 0.0.0.0 for a physical device
npm test                # vitest
```

CI (`.github/workflows/ci.yml`) runs lint, format:check, and typecheck on PRs to `main`. All three
must pass.

## Frontend (`app/`)

- **Expo Router**, file-based. Route groups: `(auth)`, `(onboarding)`, `(tabs)` (home / explore /
  create / profile). Other stacks: `event/[id]`, `org/[id]` (public profile _and_ management
  console), `org/register`, `user/[id]`, `profile/*`, `settings/*`, `view-all`, `notifications`.
- **Root layout** (`app/_layout.tsx`) mounts `QueryClientProvider` → `OnboardingProvider` →
  `ThemeProvider` → animated splash → `ThemedStack`. One `QueryClient`, 30s `staleTime`, `retry: 1`.
- **Styling**: NativeWind/Tailwind. Every colour is a `lhl*` token backed by a CSS variable in
  `app/globals.css` (`:root` light, `.dark` dark) — never hardcode `bg-white` or hex values, or dark
  mode breaks. `darkMode: 'class'` so the Settings toggle can drive it.
- **Fonts**: each weight is its own registered family (RN can't pick a weight off a variable font).
  Use `font-roboto`, `font-roboto-medium`, `font-roboto-semibold`, `font-roboto-bold` — not
  `font-bold` paired with a family.
- **Server state**: TanStack Query only. Query keys are centralized in `app/lib/queryKeys.ts`
  (`events`, `saved`, `notifications`, `user`, `settings`, `org`, `feed`) — add keys there, never
  inline, so the hierarchical `invalidateQueries` prefixes keep working.
- **API client**: `app/lib/api.ts` wraps fetch (auth header, JSON parse, `ApiError`). Unreachable
  server surfaces as `ApiError` with `status: 0` / `isNetworkError`.
- **Session**: `app/lib/session.ts`. JWT in SecureStore (keychain / Android encrypted store), falls
  back to `sessionStorage` on web. Caches an `onboardingComplete` flag so cold start can route
  without a network call; the server's `users.onboarding_completed` is the source of truth.
- **Context**: `ThemeContext`, `OnboardingContext`, `CreateEventContext` (the multi-step create-event
  wizard).

### Backend URL selection (`app/config/api.ts`)

The default in **dev and prod** is the deployed Worker
(`https://loop-db.longhorn-developers.workers.dev`), which writes to the **production database**.
A fresh clone therefore needs no wrangler, no D1, no Cloudflare account.

- `EXPO_PUBLIC_USE_LOCAL_API=1` → local `wrangler dev` on port 8787 (host derived from the Expo host
  URI so a physical device works).
- `EXPO_PUBLIC_API_BASE_URL=...` → exact URL; wins over both. In release builds only `https://`
  overrides are honoured.

`EXPO_PUBLIC_*` is inlined at build time — after editing `.env` you must restart with
`npx expo start --clear` _and_ reload the app. The chosen base URL is logged at startup.

## Backend (`server/`)

Hono on Cloudflare Workers. Entry: `server/src/worker.ts`.

**Bindings**: `DB` (D1 `loop-db`), `EVENT_IMAGES` (R2), `AI` (Workers AI embeddings), `VECTORIZE`
(`loop-event-tags` index), plus `JWT_SECRET` / email provider secrets. `AI` and `VECTORIZE` have no
local emulation, so `[env.local]` in `wrangler.toml` deliberately omits them — every call site guards
on the binding being absent. Bindings are _not_ inherited by named environments; anything added at
top level that's needed at request time must be repeated under `env.local`.

**Routes** (`server/src/routes/*.worker.ts`, mounted in `worker.ts`):

| Mount            | File                      | Notable endpoints                                                                                                                                                                    |
| ---------------- | ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Longhorn-Developers/Longhorn-Loop](https://github.com/Longhorn-Developers/Longhorn-Loop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
