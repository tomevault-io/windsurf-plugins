---
trigger: always_on
description: Persistent repository context for coding agents. `CLAUDE.md` is a symlink
---

# VelaChess — Agent Guide

Persistent repository context for coding agents. `CLAUDE.md` is a symlink
to this file.

## Start here

1. This file — invariants and conventions, all of it.
2. `docs/how-to/verify-a-change.md` — what "done" means, and what a green
   suite does not prove.
3. `openspec/README.md` — when a change deserves a written intent, and
   how to write one.

No MCP servers are configured, and none are needed: the work here is
files, a shell and a database. If an external tool or data source ever
becomes part of the loop, that is when MCP earns a place — not before.

## Project

VelaChess syncs your chess.com/Lichess games, derives your opening book
from your own play, detects where you leave your habitual lines, confirms
the cost with Stockfish, and turns harmful mistakes into FSRS-scheduled
drilling exercises.

## Map

The backend is Vertical Slice Architecture — Jimmy Bogard's, not a
reinterpretation. Read `docs/explanation/architecture.md` before moving
or creating backend code; `__tests__/architecture.test.ts` fails the build
when a boundary breaks. `.agents/skills/vertical-slice-architecture` is
the working procedure.

```
apps/server         HTTP composition root — the only place hono exists; thin routes invoke slices
apps/worker         queue composition root — thin consumers invoke slices
apps/web            TanStack Start SPA — behavior slices per area, no components/hooks/utils folders

libs/application    what the system does: one vertical slice per request/use case
                    accounts/{connect-account,sync-account}  games/{list-games,get-game? via db,judge-games}
                    analysis/{request-analysis,process-analysis,get-analysis,watch-analysis}
                    drills/{seed-exercises,get-drill-queue,get-next-drill,submit-answer}
                    repertoires/{extract-repertoire,list-repertoires,…}  insights/  overview/  deviations/  auth/
libs/infra          technical mechanisms, one package each
                    db (drizzle schema/migrations/shared queries/advisory lock), queue (pg-boss behind ports),
                    stockfish (@velachess/engine), observability (@velachess/logger),
                    providers (@velachess/platforms), auth (Better Auth config)
libs/chess          rules, PGN, FEN/EPD, SAN — shared by many slices and apps/web
libs/analysis       per-move classification (win%-based) — shared by process-analysis and apps/web
libs/repertoire     book building, judgment, adherence (pure)
libs/scheduler      FSRS wrapper (ts-fsrs)
libs/ui             design system: theme tokens, shadcn primitives, chess board
libs/fixtures       pure test data      libs/test-utils   shared test harness (test-only)

packages/           intentionally public packages only — empty until that is a product decision
```

## Vertical slice rules

- The unit is the request/use case, never the technical layer. New
  behavior = new slice directory under its area; the area is navigation,
  the slice is the architecture.
- A slice owns its validation, queries, errors, calculations and tests.
  A query used by one slice lives in the slice; multi-slice queries stay
  in `libs/infra/db` with the justification written down.
- No Controller→Service→Repository pipeline, no repository interface per
  entity, no service class per noun, no `commands/`/`queries/` buckets,
  no `core/`/`common/`/`shared/`/`utils/` dumping grounds. The
  architecture test rejects `services/`, `repositories/` etc. inside
  application by path.
- Slices do not import other slices. The documented exceptions live in
  the allowlist in `__tests__/architecture.test.ts`; extending it is an
  architecture decision recorded in `docs/explanation/architecture.md`.
- Do not extract shared code because two slices look similar — small
  duplication between slices is cheaper than coupling. Extract only for
  infra, a stable domain concept, or an intentional public package.
- Start every slice as the simplest thing (a transaction script over
  Drizzle is fine); richer patterns are earned by observed complexity.
- Different slices may use different internal patterns. Do not normalize
  them for visual consistency.

## Dependency direction

```
apps/server, apps/worker → libs/application → libs/infra + domain libs
```

Nothing under `libs/` may import from `apps/` — enforced. Application
never imports hono or pg-boss (ports only). Infra never imports
application. Worker consumers are thin delivery adapters over slices.
HTTP-shape zod stays in `apps/server` (transport translation; it is what
keeps the `hc<AppType>` client typed).

## Rules

- Explicit over clever. Optimize the number of concepts a reader holds at
  once, not function length.
- YAGNI. Functions over classes. No factory/strategy/registry patterns
  without proven need. Two implementations don't justify an abstraction.
- No magic strings — typed constants (e.g. `EXTRACTED_REPERTOIRE_NAME`).
- Frontend: `libs/ui` owns every design token; `apps/web` inherits the
  theme and never declares one. Third-party component code (shadcn, Trophy
  registry) is installed into `libs/ui` and re-exported from there.
  `apps/web/src` groups by vertical (what code does), never by technical
  type — no `components/`, `hooks/`, `utils/` folders.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [velachess/velachess](https://github.com/velachess/velachess) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
