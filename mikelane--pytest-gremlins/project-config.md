---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

pytest-gremlins is a **fast-first** mutation testing plugin for pytest. Speed is the primary
differentiator - we aim to make mutation testing practical for everyday TDD, not just overnight
CI jobs.

## Design Documents

- [North Star](docs/design/NORTH_STAR.md) - Vision, speed architecture, success metrics
- [Operators](docs/design/OPERATORS.md) - Mutation operators design and registry
- [Agent Guidelines](docs/design/AGENT_GUIDELINES.md) - TDD laws, BDD process, CI/CD, release process

## Core Architecture (Speed-First)

Four pillars drive our speed strategy:

1. **Mutation Switching** - Instrument code once with all mutations embedded, toggle via environment
   variable. No file I/O, no module reloads during test runs.

2. **Coverage-Guided Test Selection** - Only run tests that actually cover the mutated code. 10-100x reduction in test executions.

3. **Incremental Analysis** - Cache results keyed by content hashes. Skip unchanged code/tests on subsequent runs.

4. **Parallel Execution** - Distribute gremlins across worker processes. Mutation switching makes
   this safe (no shared mutable state).

## Domain Language

| Traditional Term       | Gremlin Term            |
| ---------------------- | ----------------------- |
| Original code          | **Mogwai**              |
| Start mutation testing | **Feed after midnight** |
| Mutant                 | **Gremlin**             |
| Kill mutant            | **Zap**                 |
| Surviving mutant       | **Survivor**            |

## Project Structure

```text
pytest-gremlins/
├── src/pytest_gremlins/      # Source code
│   ├── __init__.py           # Package init with version
│   ├── config.py             # Configuration loading from pyproject.toml
│   ├── plugin.py             # pytest plugin hooks
│   ├── py.typed              # PEP 561 marker
│   ├── cache/                # Incremental analysis cache
│   ├── coverage/             # Coverage-guided test selection
│   ├── instrumentation/      # AST transformation and mutation switching
│   ├── operators/            # Mutation operator definitions
│   ├── parallel/             # Parallel execution support
│   └── reporting/            # Result reporting (console, HTML, JSON)
├── tests/
│   ├── conftest.py           # Shared fixtures
│   ├── benchmark/            # Benchmark tests
│   ├── cache/                # Cache domain tests
│   ├── config/               # Config domain tests
│   ├── coverage_module/      # Coverage domain tests
│   ├── instrumentation/      # Instrumentation domain tests
│   ├── operators/            # Operator domain tests
│   ├── parallel/             # Parallel domain tests
│   ├── plugin/               # Plugin integration tests
│   └── reporting/            # Reporting domain tests
├── features/                 # Gherkin scenarios
├── docs/
│   └── design/               # Design documents
├── .github/workflows/        # CI/CD
├── pyproject.toml            # Project config (uv, ruff, mypy, pytest)
├── tox.ini                   # Multi-version testing
└── .pre-commit-config.yaml   # Pre-commit hooks
```

## Build Commands

```bash
# Install dependencies
uv sync --dev

# Run small tests only (fast, always safe)
uv run pytest tests -m small

# Run small + medium tests (PR checks)
uv run pytest tests -m "small or medium"

# Run all tests
uv run pytest

# Type checking (strict)
uv run mypy src/pytest_gremlins

# Linting
uv run ruff check src tests

# Formatting
uv run ruff format src tests

# All checks (what pre-commit runs)
uv run pre-commit run --all-files

# Multi-version testing
uv run tox

# Build docs
uv run mkdocs build --strict

# Run doctests
# Filesystem isolation checks are disabled: doctests can't carry pytest markers,
# so pytest-test-categories flags pdb reading its config file as a violation.
uv run pytest --doctest-modules src/pytest_gremlins --override-ini="test_categories_enforcement=disabled"
```

## CI Caching

pytest-gremlins uses a two-layer cache in CI:

1. **CI platform cache** (outer) — stores `.gremlins_cache/` between jobs
2. **IncrementalCache** (inner) — per-gremlin results by content hash

**Critical rules for CI cache configuration:**

- Use `hashFiles` on source + test files + `pyproject.toml` (NOT commit SHA)
- Always use `restore-keys` prefix fallback (so a file change gets a warm cache)
- Save the cache with `if: always()` — threshold check failures must NOT discard the cache
- Use split restore/save steps (not combined `actions/cache`) so `if: always()` works

**Why `if: always()` matters:** A failing threshold check exits the job non-zero. CI
skips subsequent steps by default, so the cache save never runs and the next job starts
cold. `if: always()` overrides that — the save runs even when the score gate fails, so
the next run only re-tests what changed.

## TDD Laws (STRICTLY ENFORCED)

1. **No production code without a failing test**
2. **Only enough test code to fail**
3. **Only enough production code to pass**
4. **Refactor only when green**

## Code Style

- **Quotes:** Single quotes
- **Line length:** 120 characters

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mikelane/pytest-gremlins](https://github.com/mikelane/pytest-gremlins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
