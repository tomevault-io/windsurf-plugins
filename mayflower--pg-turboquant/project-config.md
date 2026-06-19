---
trigger: always_on
description: This repository is operated as a **tests-first, prompt-driven implementation plan** for `pg_turboquant`.
---

# AGENTS.md

This repository is operated as a **tests-first, prompt-driven implementation plan** for `pg_turboquant`.

## Read order before doing any work

1. `README.md`
2. `docs/PRD.md`
3. every ADR in `docs/adrs/` in lexical order
4. the current prompt in `prompts/`
5. this file again before making changes

## Mission

Implement `pg_turboquant`, a PostgreSQL custom index access method for compact ANN retrieval over `vector` and `halfvec`, by following the prompt sequence and the accepted ADRs.

## Hard rules

1. **Tests first.** Every prompt starts by adding failing tests.
2. **Small increments.** Do only the requested slice.
3. **Do not violate accepted ADRs silently.** If a decision must change, add a new ADR and reference it here.
4. **Do not add internal heap reranking.** Exact reranking belongs in SQL, not in the access method.
5. **Do not assume giant batches per page.** Lane count must come from page budget.
6. **Do not persist dense transform matrices in v1.**
7. **Do not implement online router refresh in v1.**
8. **Keep pgvector interactions behind a small compatibility layer.**
9. **Scalar correctness is the oracle.** SIMD must prove parity.
10. **Use generic WAL in v1.**

## Working conventions

- Prefer C modules with narrow responsibilities.
- Keep page-format code isolated from codec code.
- Keep pgvector compatibility code isolated from the rest of the project.
- Prefer deterministic seeded fixtures in tests.
- Document any approximation tolerance directly in the test.
- When a prompt is complete, update the tracker below before stopping.
- Never mark a prompt `DONE` without recording the tests you ran.

## Expected repository layout once implementation begins

- `src/` — extension C sources
- `sql/` — extension migration/install scripts
- `test/sql/`, `test/expected/` — `pg_regress`
- `tests/unit/` — unit tests
- `t/` — TAP tests
- `scripts/` — bootstrap and benchmark helpers
- `third_party/pgvector/` — pinned pgvector source dependency or compatible bootstrap target

## Required test commands

These are the canonical commands to reference in tracker notes once the environment exists:

- `make`
- `make install`
- `make unitcheck`
- `make installcheck`
- TAP wrapper command defined by the repo
- benchmark smoke command from `scripts/`

## Status values

Use one of:

- `TODO`
- `IN PROGRESS`
- `DONE`
- `BLOCKED`

## Tracker update protocol

When starting a prompt:
1. set its status to `IN PROGRESS`
2. add the date
3. add a short note about the branch or working tree intent

When finishing a prompt:
1. set its status to `DONE`
2. record commands actually run
3. record any known follow-up risk or debt

When blocked:
1. set status to `BLOCKED`
2. record the exact blocker
3. do not continue to the next prompt until resolved

## Current modernization order

The repository is no longer in the original "run every pack in lexical order" phase.

Current intent:

1. Treat `prompts/` plus `prompts/pg_turboquant_corrections_pack/` as the baseline implementation history that has already been modernized where needed.
2. Use `prompts/pg_turboquant_post_v1_prompt_pack/` as the active modernization lane for the storage, planner, maintenance, query-helper, and benchmark contracts.
3. Execute the post-v1 pack in the agreed working order recorded by this tracker, which is currently not purely lexical.
4. Do not start `prompts/pg_turboquant_rag_benchmark_pack/` until the post-v1 modernization lane is in a usable state for that work.

Current agreed post-v1 execution order in practice:

- `00` through `07`
- `09`
- `10`
- `11`
- `08` now, before returning to later post-v1 prompts
- `12+` after the WAL/layout modernization and related benchmark contracts are stable

Testing policy for this phase:

- The active prompt's tests still come first.
- Broader regression suites must be kept green against the current post-v1 contract.
- Do not roll code back to satisfy stale expectations from earlier prompt packs when those expectations have been superseded by the post-v1 modernization work. Update outdated tests and golden files to the newer contract instead.

## Snapshot audit note

As of `2026-03-28`, the post-v1 tracker status has been independently re-audited against the filesystem snapshot, not just prior tracker claims.

- This audit is based on repo-local code, SQL, test, TAP, CI, and benchmark artifacts.
- This workspace snapshot does **not** include a `.git/` directory, so the audit cannot prove historical branch state or recover work lost in crashed sessions.
- For post-v1 prompts `00` through `13`, there is repo-local evidence for the claimed implementation state.
- For post-v1 prompts `14` through `17`, there is no repo-local implementation evidence matching the prompt goals in this snapshot.
- For post-v1 prompt `18`, benchmark groundwork exists, but the final comparative report contract is not complete in this snapshot.

Repo-local evidence used for the post-v1 `00` through `13` audit:

- `00`: `scripts/benchmark_suite.py`, `tests/test_benchmark_suite.py`, `t/004_benchmark_smoke.pl`, `t/005_benchmark_suite_ci.pl`
- `01`: `test/sql/metric_fidelity.sql`, `tests/unit/test_smoke.c`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mayflower/pg_turboquant](https://github.com/mayflower/pg_turboquant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
