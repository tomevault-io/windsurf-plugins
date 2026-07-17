---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

**BookMap** — AI-powered MVP where users input a topic and get an ordered reading path of real books (Gemini generates, Google Books verifies), then track progress and notes. Portfolio project: ship reliable and simple, not feature-complete.

Companion rule files (read before proposing features or refactors):
- `AGENTS.md` — coding standards, DB schema mental model, and the MVP "No-Go Zone" (reject: social features, rich text editors, graph visualizations, infinite scroll/complex caching).
- `copilot-instructions.md` and `.ai/project-definition.md` — MVP scope, explicit non-goals, tech constraints.
- `openspec/` — OpenSpec change-proposal workflow is used for some features; check specs before large UI changes.

## Commands

pnpm is the ONLY package manager (never npm/yarn/bun).

```bash
pnpm dev          # dev server (localhost:3000)
pnpm build        # production build
pnpm lint         # eslint
```

There is **no test setup** (no test runner, no test script, no test files).

## Tech stack (strict — do not introduce alternatives)

Next.js 16 (App Router) · TypeScript strict (no `any`) · Supabase (Auth + Postgres + RLS) · Tailwind CSS 4 + shadcn/ui (new-york style, `src/components/ui/`) · Vercel AI SDK (`ai` v7) + `@ai-sdk/google` (Gemini) · Google Books API · TanStack Query · Zustand · Zod 4 · react-hook-form · sonner (toasts).

## Architecture — layered flow

```
UI components (client)
  → src/app/actions/**        "use server" actions: validation, auth check, orchestration
    → src/controllers/**      read-composition / cross-service orchestration (NOT a mandatory hop)
      → src/services/**       business logic, external API calls
        → src/infrastructure/repositories/SupabaseRepo.ts   all DB reads/writes
          → src/lib/supabase/{server,client,proxy}.ts       Supabase SDK clients
```

Key nuance: **controllers are not always in the loop.** Mutations often go Action → Service directly (`processAndSaveMap`, `updateBookStatus`, `deleteMapById`). Controllers are used for (a) RSC page-level read composition (`mapController.getMapsData`/`getMapData` from pages) and (b) auth/profile flows (consistently Action → Controller → Service). Check sibling actions before deciding whether a new feature needs a controller function.

### Directory roles (`src/`)

- `app/` — routes (flat, no route groups) + `app/actions/{IA,auth,books,maps,profile}/` server actions. The only API route is `app/auth/callback/route.ts` (OAuth code exchange).
- `proxy.ts` — Next.js 16's renamed `middleware.ts`. Delegates to `lib/supabase/proxy.ts` (`updateSession`): refreshes session cookies and redirects unauthenticated users to `/auth` (public paths: `/` and `/auth/*`).
- `domain/` — all cross-layer types: `db/db_types.ts` (generated Supabase `Database` type, source of truth), `entities/models/models.ts` (app types derived from it + composite `Bookmap`/`MapItem`), `schemes/` (Zod: `aiMapResponseSchema` for Gemini output, `NoteContentScheme` for notes). Never redefine DTOs in pages/services.
- `infrastructure/` — `SupabaseRepository` class (factory `getSupabaseRepo()`) + `querys/getMapQuerys.ts` (reusable `MAP_DETAILS_SELECT` select string with `QueryData`-derived types).
- `services/` — business logic; external APIs are called inline here (no wrapper clients): Google Books via `fetch` in `books/bookService.ts` (checks `response.ok`, retries transient 5xx/429 with backoff, fails fast on 4xx), Gemini via AI SDK in `IA/maps/bookMapGenService.ts`. AI model ids live in `services/IA/config.ts` (`AI_MODELS` — single source of truth; never hardcode a model inside a service).
- `stores/` + `providers/` — `useMapStore` (Zustand) holds the currently-viewed map. `MapStoreProvider` is not a React Context: it's a client component that pushes RSC-fetched data into the global store via `useEffect`. Only one map can be "active" at a time. `TanStackProvider` mounts React Query at the root layout.
- `hooks/querys/` — thin `useMutation` wrappers around server actions (throw on `!result.success` so React Query sees errors).

### Bookmap generation flow (the core feature)

1. `GenerateForm` → action `getBookMap` → `bookMapGenService.ts`: `generateText` with `google(...)` (model id from `services/IA/config.ts` → `gemini-3.5-flash`) and structured output validated by `aiMapResponseSchema`. Prompt in `services/IA/maps/prompt.ts`.
2. Result → action `processAndSaveMap`: adapt via `lib/adapters/ai-adapter.ts` → `bookController.getProcesedBooks` verifies each book against Google Books (ISBN_13 > ISBN_10 > first identifier; **book discarded if no ISBN found** — never hallucinate books) → `mapService.createMap` → `SupabaseRepo.createMap` (insert `maps` → upsert `books` on `isbn` → insert `map_items`).
3. Cover fallback chain (`coverFallbackService.ts`): Google Books thumbnail → Open Library covers (HEAD-checked) → placehold.co deterministic placeholder. Never returns null.

`services/IA/mocks/` is dead code (not imported by the live flow; no mock toggle exists).

### Auth


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EleanQuintero/bookmaps](https://github.com/EleanQuintero/bookmaps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
