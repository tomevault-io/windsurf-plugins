---
trigger: always_on
description: - This repository contains a plugin at the repo root and a full upstream-style AstrBot source tree in `AstrBot_src/`.
---

# AGENTS.md

## Scope

- This repository contains a plugin at the repo root and a full upstream-style AstrBot source tree in `AstrBot_src/`.
- Treat `AstrBot_src/` as the authoritative reference for runtime conventions, test commands, linting, and dashboard workflows.
- Root-level plugin files include `main.py`, `config.json`, `metadata.yaml`, `_conf_schema.json`, and docs in Chinese.
- There is an existing upstream agent guide at `AstrBot_src/AGENTS.md`; keep this root guide aligned with it when editing both areas.
- No Cursor rules were found in `.cursor/rules/` or `.cursorrules`.
- No GitHub Copilot instructions were found in `.github/copilot-instructions.md`.

## Repository Layout

- Root: plugin package `astrbot_plugin_DynamicPersona` implemented in `main.py`.
- `config.json`: sample plugin config and schema-driven defaults.
- `metadata.yaml`: plugin manifest used by AstrBot.
- `AstrBot_src/`: main Python application, tests, CLI, dashboard, scripts, and Makefile.
- `AstrBot_src/dashboard/`: Vue 3 + Vite + TypeScript dashboard.
- `AstrBot_src/tests/`: pytest suite with unit, integration, and feature tests.

## Environment Expectations

- Prefer `uv` for Python environment management inside `AstrBot_src/`.
- Install dev dependencies with `uv sync --group dev`.
- `AstrBot_src/pyproject.toml` declares `requires-python = ">=3.12"`; use Python 3.12 unless you have a strong reason not to.
- Python code is linted/formatted with Ruff and upgraded with `pyupgrade` via pre-commit.
- Frontend work uses `pnpm` in `AstrBot_src/dashboard/`.
- The app commonly expects local directories such as `data/plugins`, `data/config`, `data/temp`, and `data/skills`.

## Core Commands

- Sync Python deps: `uv sync --group dev`
- Run AstrBot from source tree: `uv run main.py`
- Run AstrBot CLI: `uv run astrbot run`
- Format Python: `uv run ruff format .`
- Lint Python: `uv run ruff check .`
- Check formatting only: `uv run ruff format --check .`
- Run all tests: `uv run pytest -q`
- Run verbose full test flow: `uv run pytest --cov=. -v -o log_cli=true -o log_level=DEBUG`
- Run recommended local PR check: `make pr-test-neo`
- Run full local PR check: `make pr-test-full`
- Run faster repeated full check: `make pr-test-full-fast`

## Single-Test Commands

- Run one test file: `uv run pytest tests/test_dashboard.py -q`
- Run one test function: `uv run pytest tests/test_dashboard.py::test_neo_skills_routes -q`
- Run one unit test file: `uv run pytest tests/unit/test_astr_agent_tool_exec.py -q`
- Run one parametrized or async test by node id: `uv run pytest 'tests/unit/test_astr_agent_tool_exec.py::test_collect_handoff_image_urls_filters_supported_schemes_and_extensions' -q`
- Run tests by keyword: `uv run pytest -k handoff -q`
- Run only blocking subset: `uv run pytest --test-profile blocking -q`
- Add extra args to scripted PR env runs through `PYTEST_ARGS`, for example: `PYTEST_ARGS='tests/test_dashboard.py::test_neo_skills_routes -q' ./scripts/pr_test_env.sh --profile neo --skip-smoke`

## Dashboard Commands

- Install dashboard deps: `pnpm --dir dashboard install`
- Start dashboard dev server: `pnpm --dir dashboard dev`
- Build dashboard: `pnpm --dir dashboard build`
- Preview dashboard build: `pnpm --dir dashboard preview`
- Type-check dashboard: `pnpm --dir dashboard typecheck`
- Lint dashboard: `pnpm --dir dashboard lint`

## Validation Workflow

- For Python-only changes, usually run `uv run ruff format .`, `uv run ruff check .`, and the smallest relevant pytest target.
- For dashboard-only changes, usually run `pnpm --dir dashboard typecheck` and `pnpm --dir dashboard build`.
- For cross-cutting changes, prefer `make pr-test-neo` first, then `make pr-test-full` if the change touches core execution paths.
- `scripts/pr_test_env.sh` also performs a smoke test against `http://localhost:6185` unless skipped.
- Full profile builds the dashboard automatically unless `--no-dashboard` is passed.

## Python Style

- Follow Ruff defaults configured in `AstrBot_src/pyproject.toml`.
- Line length target is 88.
- Import order matters; Ruff uses `I` rules, so keep imports sorted and grouped.
- Prefer module-level imports at the top of the file unless a local import is needed to avoid heavy startup cost or circular imports.
- Use double quotes in Python where formatting tools prefer them.
- Keep functions and methods small and direct; the codebase favors pragmatic, readable control flow over abstraction-heavy patterns.
- Prefer explicit early returns for guard conditions.
- Keep comments sparse; add them only when behavior is non-obvious.
- Use English for all new comments, even if nearby code or docs are in Chinese.

## Python Types

- Use modern built-in generics such as `list[str]`, `dict[str, Any]`, and `str | None`.
- Match the existing codebase style: type annotate public functions, important locals, fixtures, and return values when it improves clarity.
- Pyright runs in `basic` mode, so favor useful annotations without overengineering types.
- Use `Path` objects for filesystem work where practical.
- Follow the existing project guidance: prefer `pathlib.Path` over stringly-typed paths for new code.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KirisameLonnet/astrbot_plugin_DynamicPersona](https://github.com/KirisameLonnet/astrbot_plugin_DynamicPersona) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
