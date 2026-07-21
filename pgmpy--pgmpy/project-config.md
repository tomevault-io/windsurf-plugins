---
trigger: always_on
description: This file provides guidance for all AI agents when working with code in this repository.
---

# Agent Instructions - pgmpy

This file provides guidance for all AI agents when working with code in this repository.

## Project Overview

pgmpy is a Python library for Causal AI using DAGs, Bayesian Networks, and related models. It provides tools for causal
discovery, parameter estimation, inference, causal identification, and causal effect estimation.

## Development Setup

Requires Python >=3.10.

```bash
pip install -e .[tests]        # bash
pip install -e ".[tests]"      # zsh
pre-commit install             # set up git hooks
```

For PyTorch/GPU support: `pip install -e .[torch,tests]`. Other extras: `[optional]` (litellm for LLM-based features,
xgboost, plotting), `[docs]`, and `[all]`.

## Common Commands

pgmpy uses `pytest` for testing and `pre-commit` for code quality. Here are some common commands:

```bash
# Run all tests (add -n auto to run in parallel via pytest-xdist)
pytest -v pgmpy

# Run a specific test file
pytest -v pgmpy/tests/test_models/test_DiscreteBayesianNetwork.py

# Run a specific test class or method
pytest -v pgmpy/tests/test_models/test_DiscreteBayesianNetwork.py::TestBayesianModelCreation::test_add_cpds

# Linting/formatting (pre-commit runs ruff check --fix and ruff format; config in pyproject.toml)
pre-commit run --all-files

# Run docstring examples as doctests (also run in CI)
pytest --doctest-modules --ignore=pgmpy/tests pgmpy/
```

## Instructions for Tasks
1. Always make a plan before coding.
2. For behavior changes and bug fixes, prefer writing tests first (Test-Driven Development) when practical. For pure
   documentation or non-behavioral refactors, add tests only if behavior is affected. Avoid adding too many tests and
   try to combine tests when possible.
3. Follow existing patterns in the codebase. Always check for similar implementations before creating new ones. When
   adding a new causal discovery algorithm, CI test, structure score, metric, dataset, or example model, start from
   the matching template in `devtools/extension_templates/`. When changing a base-class API, update the affected
   templates as well.
4. When implementing or suggesting changes, also look into how similar packages — both Python (e.g., causal-learn,
   DoWhy, scikit-learn for API conventions) and R (e.g., bnlearn, pcalg, dagitty) — approach the same problem, and
   combine that information into the suggestion or design. Where possible, use these packages as reference
   implementations to verify the correctness of pgmpy's implementation.
5. Use type hints and docstrings for new or modified public methods.
6. Run the smallest relevant `pytest` target after changes to ensure correctness. Broaden to larger suites as needed.
   Run `pre-commit` when it is available in the environment.
7. Avoid redundant checks in the code. For example, if a variable is always expected to be a list, do not add checks to
   verify that it is a list. Try to avoid adding `try`/`except` blocks unless absolutely necessary. If you need to add error
   handling, make sure to be explicit about the expected exceptions and handle them appropriately.
8. Check if the method that you are using has a deprecation warning. If it does, try to use the recommended alternative
   instead of the deprecated method.
9. Preserve backwards compatibility unless the user explicitly requests or approves a breaking change. For migrations
   and refactors, prefer adding new APIs alongside compatibility shims before removing old paths.
10. If a required command or dependency is unavailable in the environment, state that explicitly and use the best
    available validation instead of silently skipping verification.
11. For code that supports multiple backends or optional dependencies, preserve existing `numpy` and `torch` behavior
    where applicable, and guard optional-dependency tests appropriately.
12. Never run `git commit` or `git push`. Leave all changes in the working tree — the user always reviews and commits
    manually. Suggesting logical commit units or a commit message is fine.

## Code Style

- **Formatter + linter**: ruff (`ruff format`, `ruff check --fix`); import sorting via ruff's isort rules
- **Config**: `[tool.ruff]` in `pyproject.toml` (line length: 120)
- Pre-commit hooks auto-run on commit after `pre-commit install`
- Prefer linear methods that read top to bottom over splitting logic into many small functions, so the whole algorithm
  can be followed in one place. Only factor out a helper when it is called from multiple locations.
- Do not add comments that explain your own reasoning for a change (that belongs in the commit/PR message). Comment
  only what the code cannot express itself.

## Standard Workflow
```bash
# 1. Create a feature or bugfix branch (human workflow; optional for agents
# already working in an assigned branch/worktree)
git checkout -b feature/your-feature dev

# 2. Make changes with tests
# ... edit files ...

# 3. Verify
pre-commit run --all-files
pytest -n auto pgmpy

# 4. Leave the changes uncommitted — the user reviews and commits manually
```

## Architecture

### Base Graph Classes (`pgmpy/base/`)
Key bases: `DAG`, `PDAG`, `MAG`, `ADMG`, `UndirectedGraph`, and `SimpleCausalModel` (a `DAG` built automatically from

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pgmpy/pgmpy](https://github.com/pgmpy/pgmpy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
