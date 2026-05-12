---
trigger: always_on
description: This guide tells you where the code is, what rules to follow, and how to run, debug, and test—for the core language/runner, the apps, and new workflows.
---

# Agents Guide

This guide tells you where the code is, what rules to follow, and how to run, debug, and test—for the core language/runner, the apps, and new workflows.

---

## 1) Working on the core (WIRL language + runner)

### Code map

- **DSL grammar + parser**: `packages/wirl-lang/wirl_lang/wirl.bnf`, `packages/wirl-lang/wirl_lang/wirl_parser.py` (public API: `parse_wirl_to_objects`)
- **Runner**: `packages/wirl-pregel-runner/` — CLI entry: `python -m wirl_pregel_runner.pregel_runner` and graph builder: `wirl_pregel_runner/pregel_graph_builder.py`
- **Example DSLs + tests**: `packages/wirl-pregel-runner/tests/` and `packages/wirl-pregel-runner/tests/wirls/`
- **VSCode syntax**: `extensions/vscode/` (grammar in `syntaxes/wirl.tmLanguage.json`)

### Rules to respect

- **Grammar change rule**: anything you add to the BNF must be reflected in the parser (transformer, dataclasses) so the AST supports it. Edit `wirl_parser.py` alongside `wirl.bnf`. Keep the AST stable for existing constructs (backward compatibility).
- **Runner wiring**: grammar changes are inert until the runner knows how to execute them. Extend `pregel_graph_builder.py` if the new syntax affects scheduling, dependencies, guards, reducers, etc.
- **VSCode extension**: update tokens/keywords in `extensions/vscode/syntaxes/wirl.tmLanguage.json` to keep highlighting in sync; re‑package the extension.

### WIRL language rules

When authoring or modifying `.wirl` workflows, follow these fundamental rules:

1. **Input and output parameters are mandatory**: Every workflow must declare `input` and `output` parameters. Workflows without both will not execute properly.

2. **First node must depend on an input parameter**: The first node to run must have a dependency on at least one of the workflow's input parameters. Without this dependency, the workflow will not start execution.

3. **Cycle node inputs are restricted**: Inside cycles, nodes can only use:
   - Inputs from neighboring nodes within the cycle
   - Inputs of the cycle itself

   Inputs from outside the cycle are not directly accessible and must be proxied through cycle input parameters to be available inside the cycle.

4. **Dotted notation inside cycles**: Inside a cycle, all input values must use dotted notation, even when referencing the cycle's own inputs. For example, if you're inside a cycle named `ProcessItems`, reference cycle inputs as `ProcessItems.cycleInput` rather than just `cycleInput`.

### Run, debug, test

#### Environment setup (repo root)

```bash
make workflows-setup          # .venv + core pkgs + per-workflow deps
make workflows-setup-dev      # same, dev‑editable installs
```

These targets install `wirl-lang` and `wirl-pregel-runner` editable and pick up all `workflow_definitions/**/requirements.txt`.

#### Unit tests (runner)

```bash
make -C packages/wirl-pregel-runner test
```

Example runner tests and example `.wirl` files live under `packages/wirl-pregel-runner/tests/…`. Mirror that pattern when you add new grammar/runner features.

#### Add a focused e2e test for a new syntax

1. Create a minimal `.wirl` in `packages/wirl-pregel-runner/tests/wirls/`.
2. Add a `tests/test_*.py` that calls `run_workflow` with mock functions and asserts node outputs / guard behavior.

#### Run a workflow from the repo root (no app involved)

```bash
make run-workflow \
  WORKFLOW=<dir-under-workflow_definitions> \
  FUNCS=workflow_definitions.<name>.<name> \
  PARAMS="key1=value1 key2=value2"
```

This wraps the CLI `python -m wirl_pregel_runner.pregel_runner`. Use it to sanity‑check grammar/runner changes quickly.

#### VSCode syntax package

```bash
cd extensions/vscode
npm install
npx vsce package   # then install generated .vsix in VS Code
```

### Make targets (core)

- **Root**: `workflows-setup`, `workflows-setup-dev`, `test-workflow`, `test-all-workflows`, `run-workflow`, `install-precommit`, `lint`
- **packages/wirl-lang**: `install`, `install-dev`, `lint`, `format`
- **packages/wirl-pregel-runner**: `test` (plus `lint`/`format`). Prefer the CLI shown above over the package's `make run` (the CLI path is authoritative).

---

## 2) Working on the apps (backend, workers, frontend)

### Architecture

- **Workers** are the only component that executes workflows. They poll the DB, claim a job, run WIRL via the runner, and update `workflow_runs`. See `apps/workers/workers/worker_pool.py`.
- **Backend** is a FastAPI API over the DB, not an executor. It exposes templates, run history, and run control endpoints; it writes rows for workers to pick up. Tables live under `apps/backend/backend/models.py` (`workflow_runs`).
- **Frontend** is a simple JS app served in dev via Overmind at <http://localhost:3000>.

### Code map (apps)

- **Backend**: `apps/backend/` (FastAPI app `backend/main.py`, SQLAlchemy models, Makefile). Docs at `/api/docs`.
- **Workers**: `apps/workers/` (async worker pool in `workers/worker_pool.py`, Makefile).
- **Frontend**: `apps/frontend/` (dev service started by Overmind).

### Run & test (apps)

#### Everything via Overmind (recommended for local dev)

```bash
# repo root
make workflows-setup
# set env (DB + workflows path)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madmag77/wirl](https://github.com/madmag77/wirl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
