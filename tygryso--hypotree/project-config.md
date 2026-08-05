---
trigger: always_on
description: Use when: reading eval JSONL logs as a markdown report, scoring the GO/STOP/ITERATE gate decision, listing run-ids, diagnosing why an arm won or lost, reviewing probe economy and belief-state mechanics.
---

# hypotree — Project Instructions

## Platform & Environment

- **OS:** Linux, macOS and Windows. `eval.sh` is bash-only (WSL or Git Bash on Windows); everything else runs natively on all three.
- **Languages:** Python 3.10+
- **Frameworks/Libraries:** MCP (modelcontextprotocol), Pydantic, SciPy (Beta distributions), SQLite (stdlib)
- **Eval/Plotting:** Pandas, NumPy, Matplotlib, Seaborn
- **Tooling:** uv, pytest, ruff, mypy
- **Version Control:** Git (GitHub)

## Refactoring & Code Quality

- **Migrations and backward compatibility are required.** The project is open-source and entering production. SQLite schema changes must bump the version and upgrade old databases automatically. Deprecation warnings should precede removals by at least one minor version. Never break existing belief-state databases without a migration.
- **Prefer generalization.** When implementing a feature, look for opportunities to generalize. Enhance and consolidate logic proactively rather than adding narrow one-off code.
- **Maintain inline comments.** Keep and improve inline code comments that describe *why* something exists or *what* a non-obvious block does. Add brief logic descriptions where they aid comprehension. Never reference plan phase numbers, step numbers, or plan names in comments — comments must be self-contained, descriptive, and meaningful without any external context.

## Testing & Linting Discipline

- **Always use `uv run` — never system Python or pip.** This project uses `uv` for all environment management. Commands must be prefixed with `uv run` to use the correct virtual environment.
- **Modified code requires verification.** Any modified class, function, or variable within a file requires a **full `pytest` run on that module/file** plus **`ruff` on that file**. No exceptions — the edit is not complete until both pass.
  - **Tests:** `uv run pytest <test_file> -x -q`
  - **Linter:** `uv run ruff check <file>`
  - **Format:** `uv run ruff format <file>`
  - **Type check:** `uv run mypy src/hypotree/`
- **New code requires new tests.** Any new function, class, or method must have a corresponding new `pytest` test written for it before the task is considered done. The new test must pass alongside `ruff` on the new and changed files.
- **No untested code is shipped.** If you add code and do not write a test for it, you have failed the task. Period.

### Database Isolation in Tests

- **All new unit tests involving SQLite must use the `tmp_path` pytest fixture** to create temporary isolated databases. Never mutate real workspace databases during testing.
- **Never hardcode database paths** in tests — always derive them from the fixture. The pattern is `store = HypotreeStore(tmp_path / "test.db")`.

### Event Logging in Engine Changes

- **When adding or modifying engine mutations, ensure state changes and event logs are written within the same SQLite transaction.** A state transition that is not logged is a bug — the bi-temporal history depends on every mutation having a corresponding event record.

## Plan Management Discipline

When development follows a **PLAN** markdown document (e.g. `PLAN.md`, `PLAN-*.md`), the plan file is a living artifact and must be updated in every session:

- **Changelog at the top.** Every modification to the plan must add an entry to a changelog at the very beginning of the document (or the revision log at the end). Each entry must contain:
  - **What** — a concise description of the change.
  - **When** — the date (ISO `YYYY-MM-DD`).
  - **Where** — which file(s) were modified and what was done.
- **Phase status markers.** Every phase must be marked with one of: `DONE`, `PARTIALLY_DONE`, `PENDING`, or `REVERTED`, followed by the current date (`YYYY-MM-DD`). Example: `Phase P2c — ... — DONE (2026-07-24)`. Never leave a phase without a status marker.
- **Step-level checkboxes.** Every step within a phase must be marked as `[x]` (done) or `[ ]` (not done). If a step has no checkbox, add `[ ]` for all steps that are not done. Steps that are done must be marked `[x]` immediately upon completion. Steps must be ordered sequentially and in order of implementation and execution.

## Implementation Discipline

- After completing all code changes, **always verify your work**. Re-read the entire flow to check for missed edge cases, redundant logic, or simpler approaches. If anything can be improved, make those changes immediately.
- Prefer clean, direct implementations. Do not introduce abstractions, helpers, or wrappers unless they serve an immediate, concrete need.
- **Removing something a released version exposed is a migration, not a cleanup.** Internal helpers, dead branches and unused imports go without ceremony. A schema column, a tool name, a tool parameter or a stored reason marker is part of the contract: bump the schema and upgrade in place, or deprecate for a minor version first.

## Second Review (Mandatory)

After finishing the requested work, perform a second review pass:

1. **Summarize** all assumptions and decisions made during implementation.
2. **Re-examine** the full flow:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tygryso/hypotree](https://github.com/tygryso/hypotree) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-05 -->
