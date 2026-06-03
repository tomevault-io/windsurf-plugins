---
trigger: always_on
description: This repository uses **Ruff** as the single source of truth for linting/formatting and **Pytest** (with **pytest-cov**) for tests & coverage. CI fails when these rules are violated.
---

# AGENTS.md — Coding Rules (Ruff + Pytest + SOLID)

This repository uses **Ruff** as the single source of truth for linting/formatting and **Pytest** (with **pytest-cov**) for tests & coverage. CI fails when these rules are violated.

Run locally before committing:

```bash
# Lint & format (Ruff)
pdm run ruff check --fix .
pdm run ruff format .

# Tests & coverage (adjust --cov target if needed)
pdm run pytest --maxfail=1 -q
pdm run pytest --cov=. --cov-report=term-missing:skip-covered --cov-report=xml
```

When in doubt, prefer **correctness → clarity → consistency → brevity** (in that order).

## Table of Contents

- [1) Correctness (Ruff F - Pyflakes)](#1-correctness-ruff-f---pyflakes)
- [2) PEP 8 surface rules (Ruff E, W - pycodestyle)](#2-pep-8-surface-rules-ruff-e-w---pycodestyle)
- [3) Naming conventions (Ruff N - pep8-naming)](#3-naming-conventions-ruff-n---pep8-naming)
- [4) Imports: order & style (Ruff I - isort rules)](#4-imports-order--style-ruff-i---isort-rules)
- [5) Docstrings — content & style (Ruff D + DOC)](#5-docstrings--content--style-ruff-d--doc)
- [6) Import hygiene (Ruff TID - flake8-tidy-imports)](#6-import-hygiene-ruff-tid---flake8-tidy-imports)
- [7) Modern Python upgrades (Ruff UP - pyupgrade)](#7-modern-python-upgrades-ruff-up---pyupgrade)
- [8) Future annotations (Ruff FA - flake8-future-annotations)](#8-future-annotations-ruff-fa---flake8-future-annotations)
- [9) Local ignores (only when justified)](#9-local-ignores-only-when-justified)
- [10) Tests & examples (Pytest + Coverage)](#10-tests--examples-pytest--coverage)
- [11) Commit discipline](#11-commit-discipline)
- [12) Quick DO / DON’T](#12-quick-do--dont)
- [13) Pre-commit (recommended)](#13-pre-commit-recommended)
- [14) CI expectations](#14-ci-expectations)
- [15) SOLID design principles — Explanation & Integration](#15-solid-design-principles--explanation--integration)
- [16) Configuration management — environment variables & constants](#16-configuration-management--environment-variables--constants)
- [Final note](#final-note)

---

## 1) Correctness (Ruff F - Pyflakes)

### What It Enforces — Correctness

- No undefined names/variables.
- No unused imports/variables/arguments.
- No duplicate arguments in function definitions.
- No `import *`.

### Agent Checklist — Correctness

- Remove dead code and unused symbols.
- Keep imports minimal and explicit.
- Use local scopes (comprehensions, context managers) where appropriate.
- Do **not** read configuration from `os.environ` directly outside the dedicated constants module (see section 16).

---

## 2) PEP 8 surface rules (Ruff E, W - pycodestyle)

### What It Enforces — PEP 8 Surface

- Spacing/blank-line/indentation hygiene.
- No trailing whitespace.
- Reasonable line breaks; respect the configured line length (see `pyproject.toml` or `ruff.toml`).

### Agent Checklist — PEP 8 Surface

- Let the formatter handle whitespace.
- Break long expressions cleanly (after operators, around commas).
- End files with exactly one trailing newline.

---

## 3) Naming conventions (Ruff N - pep8-naming)

### What It Enforces — Naming

- `snake_case` for functions, methods, and non-constant variables.
- `CapWords` (PascalCase) for classes.
- `UPPER_CASE` for module-level constants.
- Exceptions end with `Error` and subclass `Exception`.

### Agent Checklist — Naming

- Avoid camelCase unless mirroring a third-party API; if unavoidable, use a targeted pragma for that line only.

---

## 4) Imports: order & style (Ruff I - isort rules)

### What It Enforces — Imports

- Group imports: 1) Standard library, 2) Third-party, 3) First-party/local.
- Alphabetical within groups; one blank line between groups.
- Prefer one import per line for clarity.

### Agent Checklist — Imports

- Keep imports at module scope (top of file).
- Only alias when it adds clarity (e.g., `import numpy as np`).

### Canonical example — Imports

```python
from __future__ import annotations

import dataclasses
import pathlib

import httpx
import pydantic

from yourpkg.core import config
from yourpkg.utils.paths import ensure_dir
```

*(Replace `yourpkg` with your top-level package. In app-only repos, keep first-party imports minimal.)*

---

## 5) Docstrings — content & style (Ruff D + DOC)

Public modules, classes, functions, and methods **must have docstrings**. Ruff enforces **pydocstyle** (`D…`) and **pydoclint** (`DOC…`).

**Single-source style**: **Google-style** docstrings with type hints in signatures.

### Rules of Thumb — Docstrings

- Triple double quotes.
- First line: one-sentence summary, capitalized, ends with a period.
- Blank line after summary, then details.
- Keep `Args/Returns/Raises` in sync with the signature.
- Use imperative mood (“Return…”, “Validate…”). Don’t repeat obvious types (use type hints).

### Function Template — Docstrings

```python
def frobnicate(path: pathlib.Path, *, force: bool = False) -> str:
    """Frobnicate the resource at ``path``.

    Performs an idempotent frobnication. If ``force`` is true, existing
    artifacts will be replaced.

    Args:
        path: Filesystem location of the target resource.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [beecave-homelab/insanely-fast-whisper-rocm](https://github.com/beecave-homelab/insanely-fast-whisper-rocm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
