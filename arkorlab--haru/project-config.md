---
trigger: always_on
description: > **Note:** Claude Code loads this file via `CLAUDE.md`.
---

# Haru Development Guide

> **Note:** Claude Code loads this file via `CLAUDE.md`.

## Commands

Root scripts fan out via Turbo (`^build` ordering in [turbo.json](turbo.json)):

```bash
pnpm install
pnpm build          # tsc -p tsconfig.build.json per package, topological
pnpm typecheck      # tsc --noEmit (TypeScript 7)
pnpm lint           # oxlint --type-aware --deny-warnings, then strict ESLint 10
                    # (depends on ^build: type-aware lint resolves dist/ types)
pnpm format         # oxfmt --write (config in oxfmt.config.ts; md/yaml excluded)
pnpm format:check   # CI gate
pnpm test           # vitest everywhere (PGlite-backed DB/server suites)

pnpm db:generate    # drizzle-kit generate -> committed under packages/db/drizzle
pnpm db:push        # apply schema to $DATABASE_URL (Neon)
pnpm db:seed        # seed a fleet from a layout JSON (--config path or HARU_FLEET_LAYOUT)

pnpm schemas:generate  # regenerate committed JSON Schemas for operator configs
                       # (packages/protocol/schemas); drift-checked by a vitest test

pnpm dev --filter=@haru/server       # via turbo (^build deps first), then tsx watch
pnpm dev --filter=@haru/supervisor   # GPU-host agent; same ^build-then-watch shape
```

Run a single test file: `pnpm --filter <pkg> exec vitest run src/foo.test.ts`
(filter by name with `-t "name"`).

**Tests assume workspace deps are built.** Every package's `exports` points at
`dist/` only; `pnpm test` handles this via Turbo's `^build`, but a standalone
`pnpm --filter <pkg> exec vitest run` in a clean checkout needs a prior
`pnpm build`.

**Schema edits require `pnpm db:generate` in the same change.** The PGlite test
harness replays the committed migrations in `packages/db/drizzle/`, while
deploys use `db:push` from the schema files; CI has a drift gate
(`db:generate` then `git status --porcelain packages/db/drizzle`, which
catches modified AND brand-new untracked migrations) that fails the PR
if they diverge.

## Architecture

Haru is a GPU HAL for Active/Standby LLM inference fleets: the active domain
serves OpenAI-compatible traffic; the standby keeps vLLM in level-1 sleep
(weights in CPU RAM) and runs preemptible LoRA training in the freed VRAM.
Promotion = stop training (SIGTERM → grace → SIGKILL, never wait for a perfect
checkpoint) → verify VRAM release → wake vLLM → synthetic probe → flip the
routing pointer. See [README.md](README.md) for the full model and
[KNOWN_ISSUES.md](KNOWN_ISSUES.md) for reviewed-and-deferred work (add new
deferrals there; delete entries when fixed).

Dependency graph (enforce it when adding imports):

```
@haru/protocol (zod + node builtins; source of type truth: schemas, enums,
   │              joinUrl/fetchWithTimeout/exec/bearer-auth shared helpers)
   ├── @haru/core            pure logic, NO I/O (state tables, plans, route intent)
   │      └── @haru/db       Drizzle schema + CAS repositories (they ENFORCE the
   │                         core state tables) + PGlite/Postgres test harness
   ├── @haru/driver-skypilot / driver-skyserve   YAML renderers + `sky` CLI wrappers
   ├── services/haru-server  Hono API + reconciler + chat proxy (uses core/db/drivers)
   └── services/haru-supervisor  depends on protocol ONLY (deliberate) - anything
       shared with the server must be hoisted into @haru/protocol, never a new dep
```

### The concurrency model (read before touching db/ or reconciler/)

- **Every state transition is a single-statement compare-and-swap**
  (`UPDATE ... WHERE state IN (...) RETURNING`, row-count checked). The Neon
  HTTP driver has NO interactive transactions - `db.transaction()` typechecks
  on `HaruDatabase`, works on PGlite in tests, and **throws at runtime on
  Neon**. Never introduce it; never hold external work (supervisor HTTP,
  `sky` exec) between a read and its dependent write.
- `fleets.activeDomainId` is the single routing pointer; `switchActive` is the
  only writer and takes an optional `requireRunningOperationId` (single-
  statement EXISTS + `FOR UPDATE` guard) so a tick racing a concurrent
  timeout-failure cannot commit routing for a failed operation. That same
  statement also sets `operations.routingCommitted` on the driving operation,
  atomically with the pointer move: the `failOperation` `target_not_routed`
  guard keys off THAT column (not a `fleets`-pointer subquery) because under
  READ COMMITTED a fail blocked on the locked operation row re-checks the row's
  own columns on unblock but keeps its snapshot for correlated subqueries - so
  a pointer subquery would let a fail land on a promotion that already went
  live (the switch-commits-first race).
- `operations` has a partial unique index enforcing one in-flight operation
  per fleet; `createOperation` joins the in-flight row on conflict.
  `sourceDomainId` records the active pointer at creation - post-commit
  cleanup steps act on it, never on "the other domain".
- The reconciler is **re-entrant check-and-nudge**: one step nudge per tick,
  every executor safe to re-run, long operations converge via status polls.
  Executor outcomes and step timeouts both map into one `StepResolution`
  applied by a single CAS-then-audit path (`applyStepResolution`); only the
  tick whose advance/complete/fail CAS lands degrades domains, cleans up

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arkorlab/haru](https://github.com/arkorlab/haru) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
