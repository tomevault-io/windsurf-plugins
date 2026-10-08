---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Working with the user

**Before making any changes**, sync the repo:
```bash
git pull                                      # pull latest commits
git submodule update --init --recursive       # update submodules to pinned commits
```

The `--recursive` flag also initializes `.claude/skills/shared` (the shared Claude Code skills submodule from `github.com/ila/duckdb-claude-skills`). If that submodule fails to initialize on its own, run:
```bash
git submodule update --init .claude/skills/shared
```

When you're stuck — either unable to fix a bug after 2-3 attempts, or tempted to work around the actual problem by redefining the objective — **stop and ask the user for directions**. Explain clearly what the specific problem is (e.g., "AstToCteList produces wrong column names for a 3-way join — should I fix the AstJoinNode output ordering or adjust the CTE binding map?"). The user knows this codebase deeply and can often point you to the right solution in one sentence. Do not silently change the goal, declare something impossible, or add bloated workarounds without consulting first. We work as a team.

Always test your changes with real queries (e.g., create a table, run `PRAGMA lpts(...)` to check the SQL output, then `SET lpts_check = true` and run the query directly to verify round-trip correctness) before declaring success, not just unit tests.

**Run the full validation (`make test`, i.e. unittest + the DuckDB-corpus coverage gate) when a feature
is fully done or before pushing** — during iteration, use `make unittest` and targeted single-file runs
instead (the gate takes a few minutes).

**Only run the SQLStorm benchmark when a feature is fully done or before pushing.** Do not run it during iterative development — it takes several minutes even at the smallest scale factor. Run it like this:
```bash
build/release/extension/lpts/lpts_sqlstorm_benchmark --tpch_sf 0.001 --timeout 10
```
The benchmark runs all 17036 SQLStorm TPC-H queries with `lpts_check` enabled (each query re-run under `SET lpts_check = true` for round-trip verification) and prints a summary (success %, not-implemented breakdown, etc.). A regression — fewer `SUCCESS` results than the previous baseline — must be investigated before pushing.

Never execute git commands that could lose code. Always ask the user for permission on those.

## Development rules

- **New features must have tests.** Ask the user whether to create a new test file or extend an existing one in `test/sql/`.
- **Never remove a failing test to "fix" a failure.** If a test fails, fix the underlying bug. Tests exist for a reason.
- **Only change `src/` and `test/` files** unless explicitly told otherwise. Do not touch CMakeLists.txt, Makefile, vcpkg.json, or any other project infrastructure files.
- **Every test file MUST verify round-trip correctness**, not just that queries run without error. Set `SET lpts_check = true` once after `require lpts`, then run the query directly — LPTS transparently compares the original and the rewritten query and raises an error on a mismatch.
- **Before implementing anything, search the existing codebase** for similar patterns or solutions. Check `BuildNode()` in `src/lpts_ast_builder.cpp` (logical operator → AST) and `FlattenNode()` in `src/lpts_ast_flattener.cpp` (AST → CTE list) as the canonical references for operator-specific logic. Reuse before reinventing.
- **Use helper functions.** Factor shared logic into helpers. Check `src/lpts_helpers.cpp` and `src/include/lpts_helpers.hpp` for existing utilities (`VecToSeparatedList`, `EscapeSingleQuotes`, etc.).
- **Never edit the `duckdb/` submodule.** The DuckDB source is read-only. All LPTS logic lives in `src/` and `test/`.
- **Add `LPTS_DEBUG_PRINT` statements** at key processing points (entry into each operator case, before/after CTE node creation, at pipeline boundaries). Use the existing macro from `src/include/lpts_debug.hpp` — it is compiled out when `LPTS_DEBUG` is 0.
- **Follow the phase split.** Operator *extraction* (reading DuckDB's `LogicalOperator` internals) belongs in `src/lpts_ast_builder.cpp`; *serialization* (SQL text) belongs in `src/lpts_ast_flattener.cpp` and `src/cte_nodes.cpp`; expression rendering belongs in `src/lpts_expression_renderer.cpp`. Do not mix phases.
- **Refuse cleanly, never emit wrong SQL.** When a query cannot be translated faithfully, raise an error whose message starts with `LPTS_<CODE>:` (use `ThrowLptsNotImplemented` from `src/lpts_helpers.cpp`). The coverage gate treats any non-`LPTS_`-formatted failure as a bug.

## What is LPTS?

LPTS (**L**ogical **P**lan **T**o **S**QL) is a DuckDB extension that takes a SQL query, reads DuckDB's internal *logical plan* for that query, and converts it back into an equivalent SQL string made of CTEs (Common Table Expressions). Each CTE corresponds to one operator from the plan.

```
D PRAGMA lpts('SELECT name FROM users WHERE age > 25');
-- WITH
-- t0_scan (t0_name) AS (
--     SELECT  "name"
--     FROM    memory.main.users
--     WHERE   (age>25)
-- )
-- SELECT  t0_name AS "name"
-- FROM    t0_scan;
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cwida/lpts](https://github.com/cwida/lpts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
