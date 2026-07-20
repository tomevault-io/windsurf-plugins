---
trigger: always_on
description: - project uses poetry for the environment & dependencies management
---

# okama — Development rules for AI agents

## environment
- project uses poetry for the environment & dependencies management
- new dependency must be added in pyproject.toml and additionally to requirements.txt
- Use interpreter in poetry env (poetry run python ...).
- Always use "poetry add" instead of "pip install"

## Test-Driven Development (TDD)
Any change to production code (new feature, bugfix, refactor, behavior change) must follow TDD: **write a failing test first, then the minimal code that makes it pass**. This overrides the default "write code, then tests" workflow.

The required workflow is the `superpowers:test-driven-development` skill. Cycle: **RED → verify RED → GREEN → verify GREEN → REFACTOR**.

Rules for this repo:
- Tests run via: `pytest -q` (or `poetry run pytest -q` if not inside the poetry shell).
- Before writing code, see the test fail for a real reason (`AssertionError` / missing function), not a typo/import error.
- For bugfix: first a test reproducing the bug, then the fix. Without a reproducing test the bug is not considered fixed.
- One test = one behavior. Test name describes the behavior meaningfully, no `test1` / `test_works`.
- Real code instead of mocks wherever possible.
- After GREEN, run the full test suite of the file/module to make sure nothing broke; output must be clean (no warnings/errors).
- Exceptions where TDD can be skipped: only by explicit user request (one-time data migration scripts, generated code, throwaway prototypes). Notebooks — partial exception: cover the library code with tests, not the notebook rendering itself.

## Post-change checklist
1) Determine whether *executable Python code* was changed, not just comments or docstrings.
2) If executable code was changed — always run tests: `pytest -q`.
3) If only comments, docstrings or Jupyter Notebooks files were changed — do not run tests.
4) If test execution reveals any failures or errors, attempt to fix them and re-run the tests.
   Do not repeat this cycle more than 2 times.
   If tests are still failing after that, stop and report the remaining issues instead of continuing.
5) Before finishing any code change (including notebooks), run `poetry run ruff check .`
   and fix every reported issue. If a warning is truly unavoidable, silence it with a
   targeted `# noqa: <CODE>` comment on the offending line and include a brief rationale.
   Never disable rules globally or use a bare `# noqa`.

## Project structure

- `okama/` — library source code
  - `api/` — data API client
  - `common/` — shared utilities and helpers
  - `frontier/` — efficient frontier optimization
  - `portfolios/` — portfolio classes
- `tests/` — pytest test suite (mirrors `okama/` layout: `asset_list/`, `helpers/`, `portfolio/`)
- `examples/` — Jupyter Notebook examples (also published to Google Colab)
- `docs/` — Sphinx documentation source

## Code search & navigation (codegraph MCP)

This repo can be indexed by the **codegraph** MCP server (a symbol graph: functions, classes, methods, call edges). It is **optional local tooling** — its config (`.mcp.json`, `.codegraph/`) is git-ignored locally and not shipped with the package. When available, prefer it for *symbol-relationship* questions over plain text search; the two are complementary. The guidance below was validated by A/B runs.

**Use codegraph when the question is about symbol relationships:**

- **who calls a symbol** — `codegraph_callers` (returns the enclosing function/method, without import/comment noise).
- **what breaks if a symbol changes** — `codegraph_impact` (transitive; depth-2 may include structural neighbours — sanity-check relevance).
- **where a symbol is defined / find a symbol** — `codegraph_search`.
- **relationship path between symbols** — `codegraph_trace`.
- **quick map of a topic** (which classes/modules are involved) — `codegraph_context`.

This pays off here because the library is class-heavy (`AssetList` / `Portfolio` / frontier hierarchies) and several modules are large (`asset_list.py`, `portfolios/core.py`, `frontier/*` run 1200–1800 lines) — jumping by symbol beats reading whole files.

**Stay on Grep/Glob or read the file when:**

- **what a function calls (`codegraph_callees`)** — the graph does **not** capture pandas/numpy method chains (`df.resample().mean()`, `.loc[...]`), so it under-reports dependencies in this data-heavy codebase. Read the body or grep instead.
- **arbitrary text** — strings, config, docstrings, ruff `# noqa` markers; and when you need *every* literal match (codegraph ranks and truncates).
- **external consumers** — the index covers this repo only; it won't show callers in okama-dash, notebooks, or downstream code.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mbk-dev/okama](https://github.com/mbk-dev/okama) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
