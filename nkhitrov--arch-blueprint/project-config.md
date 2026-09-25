---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

`arch-blueprint` is a CLI that generates architecture diagrams (PlantUML or D2) from a Python
project's module import graph. It is built on top of [`grimp`](https://github.com/seddonym/grimp),
which constructs the import graph.

## Commands

This project uses `uv` for environment and dependency management.

- Install/sync deps: `uv sync` (CI uses `uv sync --locked`)
- Run all checks (what CI runs): `uv run pre-commit run -a` (lint, format, mypy, pytest)
- Run the test suite only: `uv run pytest`
- Format: `uv run ruff format`
- Lint (with autofix): `uv run ruff check --fix src tests`
- Type-check (strict mypy): `uv run mypy ./src ./tests`
- Run the CLI against a project: `uv run arch-blueprint <project_dir> -m '<pattern>' [-f puml|d2]`
  - Example: `uv run arch-blueprint src -m 'arch_blueprint.*'`
  - Graphing **this** project is a special case: `arch_blueprint` is already in `sys.modules`
    (the CLI *is* it), so `find_spec` resolves to the running copy whatever `<project_dir>` says.
    To graph a different checkout, put it first on `PYTHONPATH` so that copy is the one running.
  - `--modules`/`-m` accepts grimp glob patterns (`pkg.*`, `pkg.**`, `pkg.*.*.models.*`).
  - `-m` can be **repeated** to graph several top-level packages at once and draw the links
    between them — useful when `<project_dir>` is a root with no `__init__.py` containing sibling
    packages (e.g. `-m 'app1.*' -m 'app2.*'`). A cross-package link is drawn only when both
    endpoints belong to the selected set — which includes a dependency *on* a package whose
    children were selected, since `pkg.*` never selects `pkg` itself.
  - `--format`/`-f` defaults to `puml`; `--no-cycle-details` hides per-module edges on cycles.
  - `--metric NAME` (repeatable) displays a metric. A node metric (`fan_in`, `fan_out`,
    `instability`) renders as a block on each node; a link metric (`edge_weight`) renders as a label
    on each connection, including cyclic ones (as `forward/backward`). An unknown name is an error,
    not a silent no-op.
- Runnable example fixture: `uv run arch-blueprint examples/project_root -m 'app1.*' -m 'app2.*' -m 'plugins.**'`
  (see `examples/README.md`) — exercises multi-root cross-links and namespace-package handling.

CI (`.github/workflows/test.yml`) has two jobs: a single-version `lint` job (`pre-commit`, skipping
pytest) and a `test` matrix running `pytest` across Python 3.9–3.14 on Linux plus one
`windows-latest` leg — that console is not UTF-8, and diagram output contains arrows, so an encoding
regression is invisible on Linux alone. Runs on push to `master` and on PRs.

### Tests

`tests/` holds the suite, one file per layer under test:

- `test_domain.py` — link aggregation, `CycleAnalyzer`, `GroupAnalyzer`.
- `test_metrics.py` — metric computation, registry routing, render plugins, `RenderPlan` validation.
- `test_renderers.py` — both renderers **in-process** (build a graph, render it, assert on the text).
- `test_source.py` — `GrimpSource`, interpreter-state hygiene, extraction.
- `test_cli.py` — exit codes, stderr messages, output encoding.
- `test_golden_puml.py` / `test_golden_d2.py` — run the CLI as a subprocess over every scenario in
  `tests/conftest.py:SCENARIOS` and assert byte-exact output against `tests/golden/<fmt>/`. When
  output changes *intentionally*, regenerate the affected golden.
- `test_golden_structure.py` — invariants the goldens must satisfy, not just their bytes: every link
  endpoint is declared, and no package wraps a class of its own name.

A hand-built `BlueprintGraph` has **empty `cycles` and `groups`** until the analyze step fills them.
A renderer test that needs either must populate them explicitly, or it will silently assert against
ungrouped nodes and plain arrows.

Fixtures: `examples/project_root` (multi-root + PEP 420 namespace package), `tests/fixtures/cyclic`
(a module cycle), `tests/fixtures/deep_ns` (single root whose link endpoints collide with node ids
and nest), `tests/fixtures/init_imports` (a package re-exporting through `__init__.py`),
`tests/fixtures/ancestor_dep` (an import of a package facade). Fixture projects are excluded from
ruff and mypy — they are analysis subjects, not code we ship.

## Git conventions

- Do **not** add self-references to commit messages or PR bodies — no `Co-Authored-By: Claude`
  trailers, no "Generated with Claude Code" lines, no mention of the assistant. Keep commit
  messages about the change only.

## Architecture

`ArchBlueprint` (`src/arch_blueprint/blueprint.py`) is a thin orchestrator: `build()` runs
everything up to rendering, `render(graph)` draws, and `run()` is the two together. The CLI uses
`build()`/`render()` separately because it must see the graph — an empty selection is a user error,
not a diagram.

1. **Source** (`extract/source.py`) — `GrimpSource` owns all grimp/`sys.path` mechanics: resolves
   every `--modules` pattern to a top-level package and builds the grimp graph (multiple roots
   supported). It handles PEP 420 namespace packages grimp can't build directly (expands them, skips

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nkhitrov/arch-blueprint](https://github.com/nkhitrov/arch-blueprint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
