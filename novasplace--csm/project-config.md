---
trigger: always_on
description: - SQLite MVP complete (Phase 3). Lint debt reduction in progress: Phase L1+L2, L3.1, L3.2, L3.3, L3.4, L3.5 done. Baseline locked at **102 warnings**.
---

## Goal
- SQLite MVP complete (Phase 3). Lint debt reduction in progress: Phase L1+L2, L3.1, L3.2, L3.3, L3.4, L3.5 done. Baseline locked at **102 warnings**.
- Phase 4 (Living State Layer) complete: experience packets, self-model, belief knowledge, advisory context-brief injection. All 4F-C requirements verified.

## Constraints & Preferences
- Each sub-phase is behavior-preserving, boring, verbatim moves first
- CSM_EMBEDDING_PROVIDER (ollama|openai), OPENAI_API_KEY, OLLAMA_HOST
- Database URL: dev/test=localhost, production=explicit flag
- CI with Postgres service container
- ESLint rules start as warnings, tighten later
- Lint warning baseline: **102 warnings** (max-warnings=102 prevents unbounded growth)
- `caughtErrorsIgnorePattern: '^_'` added to `@typescript-eslint/no-unused-vars` — catch blocks with `_err` are allowed
- `better-sqlite3` doesn't support `?NNN` format with spread `.run()` — must use anonymous `?` parameters
- SQLite schema: TEXT for timestamps/JSON/arrays/embeddings; INTEGER PRIMARY KEY AUTOINCREMENT for PKs
- PostgreSQL remains default; SQLite is adapter path, not rewrite; no vector search in SQLite MVP
- `memories.session_id` is nullable (FK on sessions, NULL bypasses it)

## Lint Debt Classification (Locked)
- **0 `no-console` warnings**: All 15 intentional console calls documented with `eslint-disable-next-line no-console` rationale
- **~102 `no-explicit-any` warnings**: Typed-debt — NOT mechanical cleanup. Requires per-module type-design work (typed DTOs, generic row mappers). Do not attempt blanket `any`→`unknown` replacement (proven to cause 55+ cascading build errors)
- **~7 `no-unused-vars` warnings**: External API generic params (`opentui.d.ts`) — skipped by design
- **`max-warnings=102`** — any new warning added to src/ will fail lint

## Progress
### Done
- **Phase 1A (Config Contract)**: `.env.example` (19 env vars), `src/config.ts` with `getEnvString/getEnvBoolean/getEnvNumber`, provider-specific env vars, mode-based DB URL, `validateAndReturnConfig()`
- **Phase 1B (Logger Foundation)**: `src/logger.ts` with levels (debug/info/warn/error) and context (session/project/turn/memoryId), `src/stats-writer.ts` updated, `src/index.ts` startup/dispose paths use logger
- **Phase 1C (Index Split)**: `src/hooks-registration.ts` (466 lines) with verbatim hooks/tools/dispose; `src/index.ts` simplified to re-exports; `src/plugin-entry.ts` removed
- **Phase 1D (CI)**: `.github/workflows/ci.yml` with PostgreSQL 14 Alpine service, typecheck/build/test steps, CSM_DATABASE_URL env var
- **Phase 1E (Mechanical Cleanup)**: Migrated 86 console calls to logger across 24 files
- **Phase 1E.1 (Lint Baseline Fix)**: Separated `lint`/`lint:src`/`lint:all`/`lint:fix` scripts. Test files excluded from lint:src. All 286 errors fixed → 0 errors.
- **Phase 1F (Hook File Split — Stabilization)**: Restored 9 deleted hook files from git. Fixed import paths, `fromSessionId` scope, redactor.ts escapes, empty blocks, `prefer-const`.
- **Phase 1G (Final Hook Registration Split)**: `hooks-registration.ts` 423→134 LOC. Created `hooks/event-hooks.ts` (125 LOC), `hooks/tool-hooks.ts` (49 LOC), `hooks/dispose-hooks.ts` (82 LOC). Plain `sessionState` object. Warnings 271→251.
- **Phase 2B (Embedding Backfill)**: `src/embedding-backfill.ts` — batch/offset/resume/rate-limited. Tool `csm_memory_backfill_embeddings`. Tests: 6/6 pass.
- **Phase 2A.1 (Dedup Detection)**: `src/dedup-detector.ts` — exact content/title + embedding ANN. Tool `csm_memory_dedup_detect` (read-only). Tests: 8/8 pass.
- **Phase 2A.2 (Safe Merge)**: `src/merge-tool.ts` — exact normalized content duplicates only. Schema: `memory_merges` table, `superseded_by`/`superseded_at`. Tool `csm_memory_merge`. Tests: 7/7 pass.
- **Lint Baseline Lock**: `max-warnings=140`. Added `caughtErrorsIgnorePattern: '^_'` to ESLint config (dropped 2 catch-var warnings). Documented `any` warnings as typed-debt, `console` warnings as intentional. Confirmed: blanket `any`→`unknown` causes 55+ build errors — do not attempt.
- **Phase 2C (Governance/Archive)**: Archive system (7,379 memories archived: 7,241 superseded + 138 tiny-junk). Governance status reports with invariant checks. Archive-aware candidate reports.
- **Phase 3A (SQLite Design)**: `docs/PHASE3A_SQLITE_DESIGN.md` — 57 files, 24 tables catalogued. Phased plan.
- **Phase 3B (Adapter Interface)**: `src/db/database-pool.ts` factory, `src/db/postgres-pool.ts` wrapper, `src/db/sqlite-pool.ts` adapter ($N→? translation, ::cast stripping, RETURNING detection). `DatabaseProvider` type. 11 new tests (9 sqlite + 2 postgres).
- **Phase 3C (Schema Bootstrap)**: `src/schema/sqlite/index.ts` — 7 tables (sessions, memories, memory_chunks, memory_merges, memory_quality_scores, memory_events, memory_recall_events). `src/schema/index.ts` early-return for sqlite. 3 smoke tests.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NovasPlace/CSM](https://github.com/NovasPlace/CSM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
