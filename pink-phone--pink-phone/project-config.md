---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## The golden rule

**Every React component must exist in Storybook (with a `*.stories.tsx`) before it is used in the app.** Build the isolated brick — its states/variants under the "felted" design system — validate it in Storybook, then compose it into screens. Do not wire a component into a page if it has no story.

## Commands

The repo has two apps: **`frontend/`** (web) and **`backend/`** (API). Frontend commands run from `frontend/`:

```bash
cd frontend
npm run storybook       # design system on :6006 (primary dev surface)
npm run dev             # the app (Vite)
npm run build           # tsc --noEmit + vite build (PWA)
npm run build-storybook # static Storybook — also the de-facto "does everything compile?" check
npx tsc --noEmit        # type-check only
npm run test            # Vitest (run once); npm run test:watch to watch
npm run screenshots     # regenerate docs/screenshots/* from built Storybook (one-time: npx playwright install chromium)
```

Tests use **Vitest + Testing Library + jsdom** (`vitest.config.ts`, setup `src/test/setup.ts` which boots i18n in `fr`); test files are `*.test.ts(x)` next to the code (pure functions like `app/mappers.ts`, `lib/`, plus component tests via RTL). There is **no linter** configured. The compile gate is `tsc` (strict mode, incl. `noUnusedLocals`/`noUnusedParameters`); run `npm run build`, `npm run build-storybook` and `npm run test` to verify changes — all must exit 0.

## What this is

PinkPhone (displayed in-app as "Pink Phone") is an intimate PWA for couples (MVP: exclusive couple, but data model is multi-partner ready). Three MVP features: **Blog** (intimate journal), **Mood** (a shared "sexual weather" indicator), **Défis** (challenges with a state machine). Distributed as a PWA, deliberately outside app stores.

Stack: React 18 + TypeScript + Tailwind v3 + `vite-plugin-pwa`, Storybook (`@storybook/react-vite`) for the frontend; a **Rust/Axum + Postgres** backend lives in `backend/` (see `backend/README.md`). The running app talks to the API (`src/api/`, `src/auth/`, `src/app/`); `src/mock/data.ts` now only feeds Storybook stories.

## Architecture

The frontend lives in **`frontend/`** (so `src/…` paths below are `frontend/src/…`); the backend lives in **`backend/`**. The frontend is layered strictly **presentational components → screens → orchestration**. Presentational pieces stay API-agnostic and live in Storybook; the orchestration layer (`src/app/`, `src/api/`, `src/auth/`) is the only place that touches the network and holds app state — it is *not* in Storybook, same category as `App.tsx`.

- `src/components/<Name>/` — reusable, presentational, controlled components. No data fetching, no global state; everything comes via props with callbacks for events. Each ships with a `.stories.tsx`. Foundations (`Surface`, `Button`, `Badge`, `Sheet`, `form/*`) are composed by feature components (`SafeMedia`, `MoodSelector`, `ReactionBar`, `VerdictPicker`, `BlogPost`, `ChallengeCard`, `PostComposer`, `ChallengeComposer`).
- `src/screens/<Name>/` — full screens (`Auth`, `Onboarding`, `Dashboard`, `Blog`, `Challenges`, `Splash`) plus `AppShell` + `BottomNav`. Presentational: they take data + handlers and map them onto components. They have stories.
- `src/app/` — stateful orchestration: `Root` (auth gate) → `SpaceGate` (load spaces / onboarding) → `SpaceApp` (the composition root) → `App.tsx` just wraps `Root` in `AuthProvider`. Each domain is a hook in `src/app/hooks/` owning its state + a **stable `refetch`** (`[spaceId]`) + mutations: `usePosts` (posts + comments), `useChallenges`, `useMoods`, `useSeen`, `useSuggestions` (challenge bank), plus `useSpaceSocket` (WS lifecycle). `SpaceApp` keeps the orchestration only: the grouped initial load + `ready` gate, the WS/resync wiring (it calls the hooks' `refetch`), confirm dialogs and sheet open/close, and derivations (partner/blind mood, "new" badge counts). Pure API→view-model conversions live in `src/app/mappers.ts`.
- `src/domain/types.ts` — **neutral layer** holding the domain enums mirrored with Rust (`ChallengeStatus`, `Intensity`, `Verdict`, `ReactionId`, `MoodId`). Both `components/` and `api/` depend on it (the arrow always points *to* the domain); the component files re-export their type for convenience. **This file is GENERATED** from the Rust consts in `backend/src/models.rs` (single source of truth, API-13): the `cargo test` `models::domain_codegen::types_ts_a_jour` regenerates it and fails if it drifted — change a status/mood/reaction value only in `models.rs`, then run `cargo test` and recommit `types.ts`. Don't hand-edit it.
- `src/types/view.ts` — presentation view-models (`Person`, `MoodSnapshot`, `PostData`, `ChallengeData`) that screens take as props and `SpaceApp`/`mappers.ts` build. The mock depends on these, not the reverse.
- `src/api/` — typed fetch client (`client.ts`, `types.ts`). Base URL from `VITE_API_URL` (default `http://localhost:8080`). `setToken` injects the JWT. Media: `uploadMedia` (multipart) and `fetchMediaObjectUrl` (authed blob → object URL, fed to `SafeMedia`'s `loader`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pink-phone/pink-phone](https://github.com/pink-phone/pink-phone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
