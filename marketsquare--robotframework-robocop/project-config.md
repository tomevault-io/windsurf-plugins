---
trigger: always_on
description: Robocop is a static code analysis tool (linter) and code formatter for
---

# Robocop

Robocop is a static code analysis tool (linter) and code formatter for
[Robot Framework](https://github.com/robotframework/robotframework). It parses `.robot`/`.resource` files using the
official Robot Framework parsing API and reports linting issues or rewrites files. Requires Python 3.10+ and Robot
Framework 5.0+.

The package lives under `src/robocop/`. The CLI entry point is `robocop = "robocop.run:main"` (Typer-based), with
subcommands `check`, `check-project`, `format`, `list`, `docs`, and `migrate`. A separate `robocop-mcp` entry point
starts an MCP server (`robocop.mcp.server:main`).

## Build, test, and lint

Uses [`uv`](https://docs.astral.sh/uv/) for environment management (`pip install uv` first).

- Run the full test suite: `uv run pytest tests`
- Run a single test file: `uv run pytest tests/linter/rules/comments/todo_in_comment/test_rule.py`
- Run a single test: `uv run pytest tests/.../test_rule.py::TestRuleAcceptance::test_rule`
- Tests run in parallel via `pytest-xdist`; mark documentation/code-example tests with the `docs` pytest marker.
- Lint: `uv run ruff check` and `uv run ruff format`
- Type check: `uv run mypy --config-file=pyproject.toml .\src\robocop\` (mypy runs in `strict` mode)
- Pre-commit runs ruff + mypy: `pre-commit install` then commits are checked automatically.

Cross-version testing (multiple Python + Robot Framework versions) is done with `nox` (sessions defined in
`noxfile.py`):
- All combinations: `nox -s tests`
- Single Python version: `nox -s --python 3.13 tests`
- Build docs: `uv run nox -s docs` (or `uv run mkdocs serve` for live preview)

Ruff is configured with `select = ["ALL"]` and a curated ignore list in `pyproject.toml`; line length is 120. Respect
the existing per-file ignores (e.g. `tests/*` skip annotations/asserts, `src/robocop/mcp/*` allows lazy imports).

## Architecture

The two core features are the **linter** (`src/robocop/linter/`) and the **formatter** (`src/robocop/formatter/`). Both
are driven through `src/robocop/run.py` (the Typer CLI) and share configuration in `src/robocop/config/`.

- **Linter** (`linter/`): `runner.py` orchestrates parsing and checker execution. Rules live in `linter/rules/*.py`,
  each module grouped by category (e.g. `comments.py`, `naming.py`, `spacing.py`, `variables.py`). Checkers subclass
  one of the base classes in `linter/rules/__init__.py`:
  - `VisitorChecker` — walks the Robot Framework AST (`ModelVisitor`).
  - `RawFileChecker` — inspects raw file lines (for things parsing can't express).
  - `ProjectChecker` / `AfterRunChecker` — run once after all files are processed.
  Diagnostics are produced via `linter/diagnostics.py`; auto-fixes via `linter/fix.py`; reports (e.g. SonarQube,
  summary tables) live in `linter/reports/`.
- **Formatter** (`formatter/`): each transformer is its own `CamelCase` file under `formatter/formatters/` (e.g.
  `NormalizeSeparators.py`, `RenameKeywords.py`). `runner.py` applies them; `aligners_core.py` holds shared alignment
  logic. `disablers.py`/`skip.py` handle opt-out directives.
- **Config** (`config/`): TOML-based configuration loaded from `pyproject.toml`, `robocop.toml`, or `robot.toml`.
  `parser.py`/`manager.py`/`schema.py` handle loading and merging; `runtime/resolver.py` resolves per-directory config.
- **MCP server** (`mcp/`): exposes lint/format over the Model Context Protocol. Optional dependency group `mcp`
  (`fastmcp`, `pydantic`). The module uses lazy (function-level) imports to isolate the optional dependency.
- **Version handling**: `version_handling.py` parses and compares Robot Framework versions so rules/tests can be gated
  by RF version (`ROBOT_VERSION`, `VersionSpecifier`). Many rules behave differently across RF 4/5/6/7.

## Key conventions

### Rule definition
Each rule is a `Rule` subclass (see `linter/rules/__init__.py`). Required attributes:
- `name` — kebab-case (e.g. `todo-in-comment`)
- `rule_id` — category prefix + number (e.g. `COM01` for comments, `SPC22` for spacing)
- `message` — supports `{placeholder}` substitution
- `severity` — `RuleSeverity.{ERROR,WARNING,INFO}`
- `added_in_version` — Robocop version the rule was introduced
- `parameters` — list of `RuleParam` for configurable options (with `converter`, `desc`, `show_type`)

The **rule docstring is the user-facing documentation** (rendered at https://robocop.dev/stable/rules_list/), so keep
examples and configuration notes there. Old rule IDs go in `deprecated_names`.

### Formatter definition
Each formatter is a class that inherits `Formatter` (a `ModelTransformer` subclass) and lives in its own file under
`formatter/formatters/` whose filename **must match the class name** (e.g. `NormalizeComments.py` →
`class NormalizeComments`). New formatters must be added to the ordered `FORMATTERS` list in
`formatter/formatters/__init__.py` — order matters, as formatters run sequentially.
- Transform the AST with `visit_*` methods (e.g. `visit_Comment`, `visit_Statement`), same as a Robot Framework
  `ModelTransformer`.
- Add `ENABLED = False` to keep a formatter out of the default run (only applied when explicitly selected via
  `--select`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MarketSquare/robotframework-robocop](https://github.com/MarketSquare/robotframework-robocop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
