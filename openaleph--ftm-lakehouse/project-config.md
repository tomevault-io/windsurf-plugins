---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Rules

1. Don’t assume. Don’t hide confusion. Surface tradeoffs.
2. Minimum code that solves the problem. Nothing speculative.
3. Touch only what you must. Clean up only your own mess.
4. Define success criteria. Loop until verified.

## Project Overview

`ftm-lakehouse` is a Python library providing data standard, archive storage, and retrieval for leaked data and document collections. It uses the FollowTheMoney data model for structured entity data and provides multi-tenant storage with support for local filesystems and S3-compatible object storage.

## Development Environment

**Important**: Always use the virtualenv at `.venv` when running commands. Either activate it or use `.venv/bin/` prefix:

```bash
# Option 1: Activate the virtual environment
source .venv/bin/activate

# Option 2: Use .venv/bin/ prefix directly (preferred for Claude)
.venv/bin/pytest -v
.venv/bin/python -m ftm_lakehouse

# Option 3: Use poetry run (if poetry is available)
poetry run <command>
```

## Common Commands

```bash
# Install dependencies (requires poetry)
poetry install --with dev --all-extras

# Full test suite: spins up the docker compose stack, runs two pytest passes
# (local+api variants, then docker variants against nginx), tears the stack down
make test

# Plain pytest – docker-variant fixtures auto-skip when the stack isn't running
poetry run pytest -v --capture=sys

# Run a single test file
poetry run pytest tests/test_unit_util.py -v

# Run a specific test
poetry run pytest tests/test_unit_util.py::test_function_name -v

# Docker compose stack (postgres + lakehouse + nginx on :8000)
make start
make stop

# Type checking
make typecheck

# Linting
make lint

# Pre-commit hooks (must be installed first)
poetry run pre-commit install
poetry run pre-commit run -a

# Run local API server (granian, port 5000, autoreload)
make api

# Build documentation (zensical, not plain mkdocs)
.venv/bin/zensical build

# Serve documentation locally
.venv/bin/zensical serve
```

## Documentation conventions

### Docstrings

Use Google-style docstrings with the canonical section order so mkdocstrings / zensical renders them consistently:

```python
def merge(self, grace_period_days: int | None = None) -> int:
    """One-line summary, imperative voice.

    Longer description, multi-line if needed. Cross-link other things with
    mkdocs-autorefs: [`LakehouseStatement`][ftm_lakehouse.model.statement.LakehouseStatement],
    [`flush`][ParquetStore.flush].

    Args:
        grace_period_days: Override ``settings.grace_period_days``. Pass ``0``
            to drop tombstones immediately.

    Returns:
        Number of statements merged.

    Yields:
        (only for generators) ``LakehouseStatement``.

    Raises:
        RuntimeError: when the dataset write fence cannot be acquired.
    """
```

- Use ``Args``, ``Returns`` / ``Yields`` (only one, matching the function), ``Raises`` – in that order. ``Example:`` / ``Examples:`` / ``Attributes:`` are the other recognized sections; anything else (``Usage:``) is not a section and renders as loose text.
- Indent the body of each section by 4 spaces under the section header.
- Backtick code/identifiers in prose (``` ``foo`` ```). Sphinx roles (``:class:`` / ``:meth:`` / ``:data:``) are **not** interpreted – they render literally – so cross-link with mkdocs-autorefs instead: ``[`display`][identifier]``.
- ``scoped_crossrefs`` is on, so ``identifier`` may be the short form when its first component is resolvable in the module's own scope (``[`merge`][ParquetStore.merge]`` inside ``storage/parquet.py``); otherwise use the full dotted path (``[`append`][ftm_lakehouse.storage.parquet.ParquetStore.append]``).
- Only identifiers mkdocstrings actually renders can be linked – check ``site/objects.inv``. Private members, undocumented classes (``EntityBuffer``, ``BaseJournalWriter``, ``RowBuffer``) and external types (``ftmq.store.lake.LakeStatement``) get plain backticks; an unresolved link warns at build time and renders as raw markdown.
- ``.venv/bin/zensical build`` must end with "No issues found" (delete ``.cache`` first – it caches the collected docstrings).

### Dashes

Use the **en-dash** ``–`` (U+2013) for parenthetical asides and ranges. Do **not** use the em-dash ``—`` (U+2014). Plain ASCII hyphen-minus ``-`` is fine for compound words and CLI flag examples.

Apply the same rule to user-facing docstrings, markdown docs, and CLI help text.

### Markdown line wrapping

Do **not** hard-wrap prose in markdown files (`.md`). Each paragraph stays on a single logical line so diffs stay clean and editors / renderers handle wrapping. This applies to docs under `docs/`, top-level READMEs, and CLAUDE.md itself. Code blocks, tables, and list markers are unaffected.

## Architecture

The codebase follows a strict layered architecture with clear separation of concerns:

```
ftm_lakehouse/
├── lake.py              # Public convenience functions (get_lakehouse, get_entities, etc.)
├── catalog.py           # Dataset config lifecycle fns + slim Catalog
│
├── model/               # Layer 1: Pure data structures (Pydantic models)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openaleph/ftm-lakehouse](https://github.com/openaleph/ftm-lakehouse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
