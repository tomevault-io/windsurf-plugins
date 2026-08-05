---
trigger: always_on
description: This file provides guidance to coding agents (Claude Code, Codex, Gemini CLI, Qwen, Copilot,
---

# AGENTS.md

This file provides guidance to coding agents (Claude Code, Codex, Gemini CLI, Qwen, Copilot,
Cursor) when working with code in this repository. It is the canonical instructions file;
`CLAUDE.md`, `GEMINI.md`, `QWEN.md`, `.cursorrules`, and `.github/copilot-instructions.md` are
symlinks to it.

PyBaMM (Python Battery Mathematical Modelling) is a battery simulation package. It is a custom
computer algebra system for writing systems of (partial) differential equations, plus a library
of battery models, parameters, solvers, and post-processing tools.

## Repository layout

This repository is a [`uv` workspace](https://docs.astral.sh/uv/concepts/projects/workspaces/) (a
monorepo) with two members under `packages/`:

- **`packages/pybamm/`** — the `pybamm` package: source in `packages/pybamm/src/pybamm/`, tests in
  `packages/pybamm/tests/`. Almost all work happens here.
- **`packages/pybammsolvers/`** — the `pybammsolvers` C++/pybind11 IDAKLU solver, with its own
  `pyproject.toml`, `ruff.toml`, and `scikit-build-core` build. `pybamm` depends on it via a
  workspace source, so a workspace sync installs it editable automatically.

Both packages release independently to PyPI. The repo root holds shared config — the workspace
`pyproject.toml` (which also carries the repo-wide Ruff and repo-review config), `noxfile.py`,
`uv.lock` — plus `docs/`. Unqualified module paths in the **Architecture** section below are
relative to `packages/pybamm/src/pybamm/`.

## Environment and commands

The project is managed with `uv`. Run all Python and tooling through `uv run` (never bare
`python -m`).

```bash
uv sync --extra all --group dev        # create/refresh the dev environment
```

Testing uses `pytest`; `unit`/`integration`/`memory` markers are assigned automatically from a
test's path (see `packages/pybamm/conftest.py`), so select suites by marker. Default `addopts` run in parallel
(`-nauto`) and treat warnings as errors.

```bash
uv run --group dev pytest -m unit packages/pybamm/tests        # full unit suite
uv run --group dev pytest -m integration packages/pybamm/tests # integration suite
uv run --group dev pytest packages/pybamm/tests/unit/test_solvers/test_solution.py   # one file
uv run --group dev pytest "packages/pybamm/tests/unit/test_plotting/test_quick_plot.py::TestQuickPlot::test_simple_ode_model"  # one test
nox -s unit | nox -s tests | nox -s integration | nox -s doctests   # sessions used in CI
```

Lint/format is Ruff via pre-commit; this is the source of truth for style.

```bash
uv run pre-commit run --all-files
```

## Architecture

PyBaMM solves models with the **Method of Lines**: a model is defined symbolically, discretised
in space, then handed to a time-stepping solver. The end-to-end pipeline (orchestrated by
`simulation.py`) is:

```
Model (symbolic) -> ParameterValues -> Geometry -> Mesh -> Discretisation -> Solver -> Solution
```

- **Expression tree** (`expression_tree/`) — the computer algebra core. Every equation is a tree
  of `Symbol` nodes (`Scalar`, `Variable`, `StateVector`, binary/unary operators, `Broadcast`,
  `Concatenation`, ...). `expression_tree/operations/` holds tree passes: conversion to
  evaluable backends (CasADi etc.), Jacobians, simplification, discretisation.
- **Models** (`models/`) — `base_model.py` defines a model as dicts: `rhs` (ODEs), `algebraic`,
  `boundary_conditions`, `initial_conditions`, `variables`, `events`. Full battery models
  (`full_battery_models/`, e.g. `lithium_ion.DFN`) are assembled by composing `submodels/`
  (particle, electrolyte, thermal, interface, ...); each submodel contributes equations for its
  physics. Presence of `rhs`/`algebraic` determines whether the system is an ODE, DAE, or
  algebraic system.
- **Parameters** (`parameters/`) — symbolic `Parameter`/`FunctionParameter` nodes; concrete
  values come from `ParameterValues`, which substitutes them into the tree. Parameter sets and
  named models are also discoverable via entry points (`dispatch/`, see `packages/pybamm/pyproject.toml`).
- **Geometry / Meshes / Spatial methods** — `discretisations/discretisation.py` walks the tree
  and replaces spatial operators with matrices and `Variable`s with `StateVector`s, using the
  `spatial_methods/` for each domain (finite volume is the default; spectral volume and
  scikit-fem elements also exist).
- **Solvers** (`solvers/`) — wrap third-party integrators (`idaklu_solver`, `casadi_solver`,
  `scipy_solver`, `jax_*`). They consume the discretised model and return a `Solution`;
  `processed_variable*.py` turns raw solver output into the named, interpolatable variables.
  **Always use `pybamm.IDAKLUSolver`** (the default for every model): it is the fastest,
  safest, most reliable, most featureful, and only actively developed solver. `CasadiSolver`,
  `ScipySolver`, and
  `CasadiAlgebraicSolver` are deprecated — never reach for them in new code, examples, or docs.
  **Don't pass `solver=` to `pybamm.Simulation` without a concrete reason — leave it bare** so
  the model's default `IDAKLUSolver` is used; only override when a specific option or behaviour
  genuinely requires it.
  **Read solution values through the interpolating call interface — `solution["Voltage [V]"](t)`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pybamm-team/PyBaMM](https://github.com/pybamm-team/PyBaMM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
