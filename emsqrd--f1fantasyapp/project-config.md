---
trigger: always_on
description: Full-stack F1 Fantasy Sports application combining React frontend and .NET backend.
---

# F1 Fantasy App Monorepo

Full-stack F1 Fantasy Sports application combining React frontend and .NET backend.

## System Overview

F1 Fantasy Sports platform where users build fantasy F1 teams, join leagues, and earn points based on real race performance.

**Architecture:**

```
React SPA (Vite) → .NET 10 Minimal API → PostgreSQL
                ↓
            Supabase Auth
```

**Tech Stack:**

- **Frontend:** React 19, TypeScript, TanStack Router, Tailwind CSS v4, shadcn/ui
- **Backend:** .NET 10 ASP.NET Core Minimal API, Entity Framework Core
- **Database:** PostgreSQL
- **Services:** Supabase (authentication), Sentry (monitoring)

## Domain & Features

### Core Concepts

- **Team** — Each user creates one team per season. Teams are subject to a budget cap; each driver/constructor has a price and the projected spend cannot exceed the cap.
- **Roster Lock** — Each race has a `LockDeadline`. Once `now >= lockDeadline`, the team can no longer be modified (drivers/constructors cannot be added or removed). The UI shows a live countdown and disables pickers when locked.
- **League** — Users create or join leagues to compete against others. Public leagues are browsable; private leagues use invite tokens. Max 15 teams per league. A team can belong to multiple leagues.
- **Scoring** — Teams earn points based on real F1 race results. See `docs/research/fantasy-rules/decisions/scoring.md` for the rules.
- **Season / Race** — Seasons map to F1 calendar years and contain ordered races (with round numbers and lock deadlines). Driver and constructor pricing is dynamic per season.

## F1 Domain

**Grid:** 22 drivers across 11 constructors. Each constructor fields exactly 2 drivers.

**Race weekends** come in two formats as it pertains to this game:

- **Standard:** Qualifying → Race
- **Sprint** (~6 per season): Sprint → Qualifying → Race

**Game rules and design decisions** are documented in `docs/research/fantasy-rules/decisions/`:

- `design-goals.md` — Player experience goals that all other decisions are evaluated against
- `format.md` — Team shape: slot counts, budget cap, composition constraints
- `rules.md` — Gameplay mechanics: transfers, captain, locking, budget, edge cases
- `scoring.md` — Point tables and scoring logic (intentionally diverges from official F1 Fantasy)
- `pricing.md` — Preseason pricing source, in-season PPM-based price movement formula

The official F1 Fantasy scoring rules are captured for reference in `docs/research/fantasy-rules/reference/f1-official-scoring.md`.

## Project Context

- **Team Size**: Solo developer
- **Development Philosophy**: Balance simplicity with proper patterns - avoid both over-engineering for scale and shortcuts that create technical debt
- **F1 Season**: The F1 season aligns with the calendar year. When referencing teams, drivers, regulations, or example data, use the current season's information first, falling back to the previous season only when current-season data isn't available.

## Claude Code Preferences

- Avoid over-engineering; keep solutions focused on the requested task
- Adhere to YAGNI philosophy
- When in doubt about approach, ask rather than proceed
- Keep solutions focused on solving the cause of a problem, not the symptom
- Use conventional commit styling for commit messages

## Feature Planning

When planning features (via plan mode or when asked to plan), organize the plan into a sequence of **self-contained commits**. Each commit is a gate — wait for user approval before moving to the next one.

- Each commit includes **both functionality and its tests** — never split implementation from tests.
- Each commit must independently pass build, lint, tests, and formatting.
- Commits should be iterative and incremental — each builds on the last.
- Order commits so earlier ones lay the foundation (e.g., data model before API before frontend).
- Keep commits focused. If a commit is doing too much, split it further.
- Write the plan to `docs/plans/` when producing a full plan.

## Testing Strategy

Three test types, each owning failure modes only it can see. Coverage is measured in failure modes, not scenarios — overlap across layers is fine when each layer catches something the others can't.

### What each layer owns

**Unit tests** — pure logic, in-process, no I/O.

- Own: branches, edge cases, error paths, mappings, calculations, validation rules.
- Goal: full combinatorial coverage of logic. Millisecond feedback.
- Dependencies: real collaborators within the process; inject clock/RNG/IDs for determinism.

**Integration tests** — one or more real boundaries crossed (DB, HTTP pipeline).

- Own: SQL correctness, EF config, migrations, serialization, auth/middleware, model binding, transaction behavior, endpoint contracts.
- Prefer the HTTP seam (`WebApplicationFactory`) over calling handlers directly — covers the pipeline for free.
- Use real Postgres via Testcontainers. **Never** EF InMemory or SQLite-for-Postgres — they diverge from prod on exactly what these tests exist to verify.
- Isolate per test (transaction rollback or truncate). No shared fixtures.
- Stub third parties you don't own (Supabase, Sentry). Don't stub your own code.

**E2E tests** — full stack through a real browser.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [emsqrd/f1fantasyapp](https://github.com/emsqrd/f1fantasyapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
