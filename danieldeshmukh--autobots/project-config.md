---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Autobots (`autobot-swarm` on PyPI) is a Python CLI that orchestrates a hierarchical coding swarm against a **target project** (which is NOT this repo). Operators run Autobots commands from the parent directory of the target project. The target project must contain six `context/` markdown files (defined in `autobots/bootstrap.py:CORE_CONTEXT_FILES`); Autobots does not create them.

The package version is `0.1.4` (see `pyproject.toml` and `autobots/__init__.py`).

## Install / Build

```powershell
# Editable install (this repo is the engine)
python -m pip install -e . --no-build-isolation

# Build distributions (CI does this in .github/workflows/publish.yml on v* tags)
python -m build

# Entry point: autobots = autobots.cli:main (also reachable as python -m autobots via __main__.py)
```

Required runtime: **Python 3.11+** (uses `tomllib`). Dependencies: `openai`, `python-dotenv`, `rich` (`tomli` only for <3.11).

`NVIDIA_API_KEY` is required for any `run`, `resume`, `engage`, or `validate-models` invocation. The CLI prompts for it on first use and writes it to the engine repo's `.env` (`autobots/cli.py:_ensure_api_key`).

## Test

```powershell
# All tests
python -m pytest tests/ -v

# One file
python -m pytest tests/test_router_contracts.py -v

# Coverage
python -m pytest tests/ --cov=autobots --cov-report=html
```

Tests use `unittest.TestCase` (not pytest fixtures) and `tempfile.TemporaryDirectory` to construct scratch workspaces — there is no `conftest.py`. Many test files are named after the development phase that introduced them (`test_phase_4_execution.py`, `test_phase_7_state.py`, etc.) plus domain tests (`test_catalog`, `test_planning`, `test_router_contracts`, `test_bootstrap`, `test_context_gen`, `test_cli_plan_args`, `test_cli_runtime`).

## High-Level Architecture

Five cooperating subsystems, each in its own subpackage:

1. **`autobots/cli.py`** — argparse-less command dispatcher. Each verb (`init`, `plan`, `run`, `resume`, `status`, `engage`, `validate-models`, `list`) has a `run_<verb>(args)` function. `main(argv)` is the entry point. **It is a known mismatch that `pyproject.toml` and `setup.cfg` both declare the entry point as `autobots.cli:main`, but the actual function is `main` — both files reference it correctly, so the entry point works. Note however that `find_endpoints.py` is referenced in `autobots/catalog.py` but does not exist in this repo; live NVIDIA catalog discovery will fall back to the bundled registry when the file is absent (see `_load_discovery_module`).**

2. **`autobots/workspace.py`** — `TargetProjectWorkspace` is the only object that touches the target project's filesystem. Allowed write roots are the fixed set `{"src", "app", "lib", "tests", "docs", "scripts", "context"}`. Critical context files (`architecture.md`, `security-auth.md`) acquire a JSON lock at `context/.autobots-locks/*.lock.json` with a 60s TTL. All writes go through `_atomic_write_text` (tmp-file + rename). Path traversal is blocked by `_resolve` checking the resolved path stays under the root.

3. **`autobots/catalog.py`** — `ClusterCatalog` defines 9 clusters (Optimus/UltraMagnus/RedAlert/Jazz/Ratchet/Perceptor/Bumblebee/Ironhide/Wheeljack), each with `ModelSpec` entries, keywords, and role. `route_with_reasoning(task_signal)` scores clusters by keyword + file-extension + role-bias signals. `select_models(cluster, signal)` picks lead/reviewer/support based on tags and the `AUTOBOTS_MODEL_SELECTION_PROFILE` env var (`balanced`/`speed`/`quality`). The catalog can merge a live NVIDIA model list (via the missing `find_endpoints.py`) with the bundled registry, or stay on the bundled fallback.

4. **`autobots/router/`** — The swarm orchestrator. `core.py:AutobotRouter.execute_phase` is the main entry: it builds a `ClusterPlan` (via `planning.ClusterPlanner`), runs four sequential stages through `stages.StageExecutor`:
   - **command** (Optimus planner) → writes a mission brief
   - **specialist** (primary cluster) → returns a JSON `{summary, implementation_notes, files: [{root, path, content}]}` — the swarm's "language"
   - **safety** (RedAlert) → returns `{status: pass|revise, summary, issues}`
   - **repair** (Ratchet) → only when review returns `revise`

   The router then runs a `_run_verification_loop` of up to `MAX_VERIFICATION_ATTEMPTS=3` (env: `AUTOBOTS_MAX_VERIFICATION_ATTEMPTS`) that re-validates and triggers Ratchet repair on failure. The result is persisted to the workspace and returned as `ExecutionResult`.

   `phases.py:PhaseReader` parses `progress-tracker.md` lines that look like `- [ ] P3 | Title | depends on: ...` or `- [~]` (in progress) or `- [x]` (complete). `utils.PayloadValidator` enforces the JSON contracts each stage must return.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DanielDeshmukh/autobots](https://github.com/DanielDeshmukh/autobots) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
