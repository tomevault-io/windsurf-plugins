---
trigger: always_on
description: This file provides guidance to Claude Code when working on the dioxide codebase.
---

# CLAUDE.md

This file provides guidance to Claude Code when working on the dioxide codebase.

## Project Overview

**dioxide** is a declarative dependency injection framework for Python that makes clean architecture simple. It combines:
- **Hexagonal Architecture API** - `@adapter.for_()` and `@service` decorators with type hints
- **Type safety** - Full support for mypy and type checkers
- **Clean architecture** - Encourages loose coupling and testability
- **Rust-backed core** - Correctness guarantees (circular dependency detection, graph validation) via PyO3

**History**: The package was renamed from `rivet_di` to `dioxide`. Legacy `rivet_di` references in the codebase should be updated to `dioxide`.

**v1.0.0 STABLE**: MLP (Minimum Loveable Product) complete. Hexagonal architecture API, lifecycle management, circular dependency detection, performance benchmarking, framework integrations (FastAPI, Flask, Celery, Click, Django, Ninja), and comprehensive testing guide all implemented. Active development continues on post-MLP features (lazy discovery, strict mode, scan planning, request scoping).

## Design Principles: The North Star

**CRITICAL**: Before making ANY architectural, API, or design decisions, consult **`docs/design-principles.md`**.

The Design Principles document is the **canonical design reference** for Dioxide. It defines:
- **The North Star**: Make the Dependency Inversion Principle feel inevitable
- **Guiding Principles**: 7 core principles (type-safe, explicit, fails fast, etc.)
- **Core API Design**: `@adapter.for_()`, `@service`, `Profile` class, container, lifecycle
- **Testing Philosophy**: Fakes at the seams, NOT mocks
- **What We're NOT Building**: Explicit exclusions list for MLP scope

**Key principle:** If design-principles.md says not to build something for MLP, don't build it. Simplicity over features.

## Quick Reference Commands

### Setup
```bash
uv venv && source .venv/bin/activate
uv sync --group dev
maturin develop
pre-commit install
```

### Testing (pytest)
```bash
uv run pytest tests/                                    # All tests
uv run pytest tests/ --cov=dioxide --cov-report=term-missing --cov-branch  # With coverage
uv run pytest tests/ -k "lifecycle"                     # Pattern match
uv run pytest tests/benchmarks/ --benchmark-only        # Benchmarks
```

### Testing (BDD/Behave)
```bash
uv run behave --tags="not @wip"                         # Regression suite (CI default)
uv run behave --tags="@issue-123"                       # Tests for specific issue
uv run behave --tags="@wip"                             # WIP tests (expected to fail)
uv run behave --tags="@issue-123" --tags="@wip"         # WIP tests for specific issue
uv run behave --dry-run --tags="@issue-123"             # Preview what would run
```

### Code Quality
```bash
ruff format python/ && cargo fmt                        # Format
ruff check python/ --fix && isort python/               # Lint Python
cargo clippy --all-targets --all-features -- -D warnings -A non-local-definitions  # Lint Rust
mypy python/                                            # Type check
```

### Building
```bash
maturin develop          # Dev build
maturin develop --release # Release build
maturin build            # Build wheel
```

### Documentation
```bash
uv sync --group docs
uv run sphinx-build -b html docs docs/_build/html
./scripts/docs-serve.sh  # Live reload server
```

## Repository Structure

```
dioxide/
├── python/dioxide/         # PUBLIC Python API
│   ├── __init__.py          # Package exports
│   ├── container.py         # Container with profile-based scanning
│   ├── adapter.py           # @adapter.for_() decorator
│   ├── services.py          # @service decorator
│   ├── lifecycle.py         # @lifecycle decorator
│   ├── profile_enum.py      # Profile class (extensible, type-safe)
│   ├── scope.py             # Scope enum (SINGLETON, FACTORY)
│   ├── exceptions.py        # Custom exceptions (incl. SideEffectWarning)
│   ├── testing.py           # Test utilities (fresh_container)
│   ├── fastapi.py           # FastAPI integration
│   ├── flask.py             # Flask integration
│   ├── celery.py            # Celery integration
│   ├── click.py             # Click CLI integration
│   ├── django.py            # Django integration
│   ├── ninja.py             # Django Ninja integration
│   ├── _registry.py         # Internal registration system
│   ├── _dioxide_core.pyi    # Rust binding type stubs
│   └── py.typed             # PEP 561 marker
├── rust/src/                # PRIVATE Rust implementation
│   └── lib.rs               # PyO3 bindings and container logic
├── tests/                   # Python integration tests
│   ├── type_checking/       # mypy type safety tests
│   ├── benchmarks/          # Performance benchmark tests
│   └── fixtures/            # Test fixtures
├── features/                # BDD/Behave acceptance tests
│   ├── steps/               # Step definitions
│   └── *.feature            # Gherkin feature files
├── examples/                # Example applications
│   ├── fastapi/             # FastAPI integration example
│   ├── flask/               # Flask integration example
│   ├── celery/              # Celery integration example

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mikelane/dioxide](https://github.com/mikelane/dioxide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
