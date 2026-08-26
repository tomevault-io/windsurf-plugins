---
trigger: always_on
description: Read this before changing the app. It describes the **current** code, not the original v1 spec.
---

# Mortang Meals — agent guide

Read this before changing the app. It describes the **current** code, not the original v1 spec.

`docs/superpowers/` is a historical design and implementation plan from 2026-08-12. It is stale: it lists web import and a recipe library as non-goals, says generate writes a new current plan, and treats Kitchen as an appliance checklist only. Do not implement from those files.

## What this is

A local, single-household meal planner. The user describes who they cook for and how they cook; the app generates a week of recipes, keeps a meal library, and derives a shopping list.

No auth, no multi-household, no hosted deploy. One Next.js process. The browser never calls an AI provider.

Success path: set up household + kitchen → pick slots on This Week → generate → cook from a card → pin keepers / swap duds / place from the library → shop from the merged list.

## Stack and commands

- Next.js 15 App Router (`src/`), React 19, TypeScript, Tailwind 4
- SQLite via `better-sqlite3` + Drizzle (queries only — schema is created in `src/lib/db.ts`, not drizzle-kit)
- Zod for AI JSON and HTTP bodies
- OpenAI SDK against xAI (`https://api.x.ai/v1`) or a custom OpenAI-compatible base URL
- Vitest. Component tests set `// @vitest-environment happy-dom`

```
npm run dev     # localhost:3000
npm test        # vitest run
npm run build
```

Env: copy `.env.example` to `.env.local` and set `XAI_API_KEY`. Optional `MORTANG_DB_PATH` overrides the SQLite file (default `data/mortang.db`, gitignored). Tests should set `MORTANG_DB_PATH` to a temp file and call `resetDbForTests()` when they open the db.

`src/app/layout.tsx` is `force-dynamic`. `next.config.ts` marks `better-sqlite3` as a server external package.

## Hard constraints

- Browser talks only to local API routes / server actions. Keys stay on the server.
- Grok key is `process.env.XAI_API_KEY` only. Never store it in SQLite. Custom-provider keys may live in settings.
- Automated tests mock the adapter (`complete`). No live model calls.
- One household. Week is Monday–Sunday (`mondayOf` in `src/lib/week.ts`). Slots are `breakfast | lunch | dinner` (21 cells).
- At most one plan has `isCurrent = 1`.
- Last good plan / meal is never replaced by a failed generate, swap, or import.
- Ingredient `quantity` is a **string** (`"1"`, `"1/2"`, `"1/4"`). Never a number. Never `0` for a used ingredient.
- Duplicate = normalized title match only (`src/meals/duplicates.ts`: lowercase, strip non-alphanumerics, collapse spaces). No fuzzy matching.
- Allergen = case-insensitive substring of an ingredient **name** (`src/meals/allergen.ts`).
- Generate and swap retry **once** on transport / invalid JSON / schema / allergen / duplicate. Then keep the previous data.
- AI traces: always record, keep last 25, redact `Bearer` tokens and `api_key=` values. Developer nav is hidden unless Settings → developer tools is on.
- Visual language lives in `src/app/globals.css` (olive / linen / paper). Match existing components; do not invent a parallel design system.

## Architecture

```
UI (server pages + client components)
    │  fetch / server actions
    ▼
HTTP handlers
    src/ai/http.ts        generate, swap, settings, traces
    src/meals/http.ts     library, place, pin, import, update, delete
    │
    ├── domain (pure, easy to test)
    │     brief, schema, allergen, duplicates, shopping-list, slot-mask, catalog
    ├── repos (SQLite)
    │     household, kitchen, prefs, meals, settings, traces
    └── adapter
          src/ai/adapter.ts  → xAI / custom OpenAI-compatible endpoint
```

Shared types: `src/lib/types.ts`. Drizzle tables: `src/lib/schema.ts`. Schema bootstrap + additive columns: `src/lib/db.ts` (`ensureSchema` + `ensureColumn`). There are no foreign keys.

Thin `src/app/api/*/route.ts` files parse JSON and call a handler. Keep logic in the handler modules so tests can call them without Next.

## Screens

| Route | Role |
| --- | --- |
| `/setup` | First-run wizard: household → kitchen checklist → slot mask. Redirect target when there is no household or no named people. |
| `/` This Week | Home. Slot picker, generate, pin all, “use what I have” ingredients, week grid, recipe flyout, library flyout. `?plan=` opens a historical plan. |
| `/meals` | Library catalog: search / filter / group, import-from-URL form. |
| `/meals/[id]` | Full recipe editor (title, why, time, method, ingredients, steps). Swap only if the meal is on the current plan. |
| `/shopping-list` | Derived list for the open plan (`?plan=` supported). Not stored. |
| `/household` | People, leftovers of diet style/notes/servings. |
| `/kitchen` | Cook prefs + appliance/method checklist. |
| `/settings` | Provider mode, base URL, model, optional custom key, web search toggle, developer tools. |
| `/developer` | Last 25 AI traces. Hidden unless the toggle is on. |

Nav: This Week, Meals, Shopping list, Household, Kitchen, Settings, optional Developer (`src/components/nav.tsx`).

Generation UX is global (`GenerationProvider` in `AppShell`): NDJSON stream. Modal on `/` and `/meals*`; compact banner on other routes. Cancel aborts the fetch.

## Data model

**Household** — one row. Name, `dietStyle` (legacy / fallback), notes, servings, people.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wmfmorgan/mortang-meals](https://github.com/wmfmorgan/mortang-meals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
