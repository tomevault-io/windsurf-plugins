---
trigger: always_on
description: Guidance for AI agents and contributors working in this repository. Read it before
---

# AGENTS.md

Guidance for AI agents and contributors working in this repository. Read it before
making changes. For module/package authoring details, defer to the existing docs
(see [Documentation](#documentation)) rather than duplicating them here.

## Project overview

This repo holds **Cognite deployment packs** — Cognite Toolkit modules
(YAML/TOML configuration) under `modules/`, plus a small set of standalone **Python**
helper scripts at the repo root that validate the package registry and build the
release archive. Most code you touch will be that Python tooling; the rest is
declarative Toolkit configuration.

The repo also includes **Qualitizer** (`modules/tools/apps/qualitizer`), a
TypeScript/React web app with its own tooling (Vite, Biome, Vitest). The Python
conventions below do not apply to it.

## Repository layout

- **`modules/`** — all deployable content (data models, transformations, functions,
  workflows, auth, dashboards, tools), organised by domain folder.
- **`modules/packages.toml`** — the registry. Every module and deployment pack must be
  registered here; nothing ships unless it is listed.
- **`validate_packages.py`** — validates `packages.toml` structure and that referenced
  module paths exist.
- **`build_packages.py`** — builds the `packages.zip` release archive.
- **`tests/`** — pytest suite for the Python tooling.
- **`.github/workflows/`** — CI (lint, type-check, validate, build, CodeQL).

## Python: how we write code

The full style contract is in [.gemini/styleguide.md](.gemini/styleguide.md). It is the
source of truth. The points below are the ones that bite most often.

- **Target Python 3.13+**, line length **120**, 4-space indentation.
- **Type hints everywhere** — every function, method, and attribute. Avoid `Any`.
  Prefer `dataclass` or Pydantic models over `dict[str, Any]`; parse files into typed
  structures.
- **Use Pydantic** for data classes where validation or parsing is involved.
- **Never** use `from __future__ import annotations`.
- **Imports at the top** of the file, grouped (stdlib, third-party, local), sorted
  alphabetically within each group, and absolute. Use `TYPE_CHECKING` for type-only
  imports.
- **Specific exceptions**, not broad `except Exception`. Return `None` / Union types for
  fallible operations and log with context (`log.warning(f"...: {var}")`).
- **No hard-coded secrets, project/cluster names, or customer identifiers** — anywhere,
  Python or YAML. Use environment variables or Toolkit variables (`{{ variable_name }}`).
- Concise docstrings with `Args:` / `Returns:` for non-trivial functions.

### Minimum code that solves the problem

Write the smallest change that solves the actual problem. **Nothing speculative** — no
"might need it later" abstractions, options, or generality that no current caller uses.
Delete dead and commented-out code rather than leaving it. If a simpler version passes
the same tests, prefer it. Always keep refactors separate from feature or fix changes —
never mix them in the same commit or PR.

## Test-driven development

We work test-first and keep tests **pragmatic and minimalistic** — cover the bug or
feature, don't over-test.

1. **Write the failing test first.** Capture the behaviour or reproduce the bug before
   touching the implementation.
2. **Make it pass** with the minimum code (see above).
3. **Refactor** with the test as your safety net.

Tests live in `tests/` and run with `pytest`. Test behaviour, not implementation
details. Choose the test shape that fits:

- **Unit tests** — pure functions and small logic units. Fast, isolated, no I/O. Use
  `tmp_path` and fixtures for file-touching helpers. This is the default for the helper
  scripts.
- **Integration tests** — exercise a script end-to-end (e.g. run `validate_packages.py`
  against a fixture `packages.toml`, or `build_packages.py` against a temp module tree)
  and assert the real output. Use these to lock in behaviour that spans several units or
  reads/writes real files.

Cover edge cases and the specific bug a fix addresses; add an example-based test that
pins the fixed behaviour so it can't regress.

```bash
# Run the whole suite
uv run pytest

# Run one file or test
uv run pytest tests/test_foundation_cicd_generator.py
uv run pytest tests/test_foundation_cicd_generator.py::test_name
```

## Python dependencies (uv)

Python packages in this repo are managed with [uv](https://docs.astral.sh/uv/). The
root `pyproject.toml` defines a **workspace** of deployable function/streamlit packages
under `modules/`, plus repo dev tools. CDF still deploys `requirements.txt` beside each
handler; those files list **direct deploy dependencies only** — packages installed on top
of the CDF Functions runtime, not the full transitive lockfile. Source of truth is
`deploy_dependencies` in `scripts/generate_uv_member_projects.py`; regenerate with
`scripts/export_deploy_requirements.py` after changing deploy deps.

```bash
# One-time / after pulling dependency changes
uv sync --group dev

# After changing any member pyproject.toml or root [dependency-groups]
uv lock
python scripts/export_deploy_requirements.py

# Run a function's tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cognitedata/library](https://github.com/cognitedata/library) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
