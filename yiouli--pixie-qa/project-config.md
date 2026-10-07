---
trigger: always_on
description: pixie-qa is a Python package and coding-agent skill for automated quality assurance of AI applications. The package is published as `pixie` and provides instrumentation, evaluation, and observability primitives that plug into LLM-powered apps.
---

# GitHub Copilot Instructions for pixie-qa

## Project Overview

pixie-qa is a Python package and coding-agent skill for automated quality assurance of AI applications. The package is published as `pixie` and provides instrumentation, evaluation, and observability primitives that plug into LLM-powered apps.

## Technology Stack

- **Python 3.11+** with type hints
- **uv** for package management, virtual environments, and builds
- **pytest** for testing
- **mypy** for static type checking
- **ruff** for linting and formatting
- **OpenTelemetry SDK** and **OpenInference** for LLM call instrumentation

## Package Structure

```
pixie/
  __init__.py
  assets/
    index.html           # compiled React scorecard (build artifact, gitignored)
  cli/
    test_command.py      # pixie test entry point
  evals/
    scorecard.py         # scorecard data models + template-based HTML generation
  instrumentation/
    __init__.py          # public API: init(), flush()
    spans.py             # ObserveSpan, LLMSpan, message/content types
    handler.py           # InstrumentationHandler ABC
    wrap.py              # wrap() API for data-object-based tracing
    processor.py         # LLMSpanProcessor (OTel SpanProcessor)
    queue.py             # _DeliveryQueue (background worker thread)
    instrumentors.py     # auto-discovers and activates OpenInference instrumentors
    py.typed

frontend/                # React scorecard SPA source
  src/                   # React components, types, styles
  package.json           # React 19, Vite 6, vite-plugin-singlefile
  vite.config.ts         # builds to ../pixie/assets/
  README.md              # frontend dev & build instructions

tests/
  README.md              # testing instructions and manual verification guide
  pixie/                 # automated tests (pytest)
    cli/
      test_test_command.py   # CLI unit tests
      e2e_fixtures/          # mock evaluators and datasets for e2e
    evals/
      test_scorecard.py
    instrumentation/
      test_spans.py
      test_context.py
      test_queue.py
      test_processor.py
      test_integration.py
  manual/                # manual testing fixtures (not run by pytest)
    mock_evaluators.py
    datasets/
      sample-qa.json     # run with: pixie test tests/manual/datasets/sample-qa.json

specs/                   # design specs and architecture docs
```

**Test file naming:**

- Test files must start with `test_` prefix
- Mirror the structure of the source code directory
- One test file per source module

---

## uv Package Management

This project uses **uv** (not pip, Poetry, or conda) for all dependency and environment management.

### Common Commands

```bash
uv sync                          # Install/sync all dependencies (creates .venv if needed)
uv add <package>                 # Add a runtime dependency
uv add --dev <package>           # Add a dev dependency
uv run pytest                    # Run a command inside the managed environment
uv run mypy pixie/               # Type check
uv run ruff check .              # Lint
uv run ruff format .             # Format
uv build                         # Build the package (sdist + wheel)
uv lock                          # Re-lock dependencies after editing pyproject.toml
```

### Rules

- **Never use `pip install` directly** — all dependency changes go through `uv add` / `uv remove` and are recorded in `pyproject.toml`.
- **Always run tools through `uv run`** to ensure the correct virtualenv is used (e.g., `uv run pytest`, not bare `pytest`, unless the venv is already activated).
- **Commit `uv.lock`** alongside `pyproject.toml` changes.

---

## Test-Driven Development (TDD) Requirements

This project follows strict TDD practices to ensure code quality and maintainability.

### 1. Test-First Development

**CRITICAL**: Always write or verify tests BEFORE implementing features or making changes.

**Development Workflow:**

1. **Understand the requirement**: Clarify what needs to be built or changed
2. **Write the test first**: Create test cases that define expected behavior
3. **Run the test**: Verify it fails (red) — this confirms the test is valid
4. **Implement the code**: Write minimal code to make the test pass
5. **Run the test again**: Verify it passes (green)
6. **Refactor if needed**: Improve code while keeping tests green
7. **Run all tests**: Ensure no regressions

### 2. Test Location and Organization

**All tests for the pixie module must be in the `tests/pixie/` directory:**

```
pixie/
  instrumentation/
    spans.py
    handler.py
    wrap.py
    processor.py
    queue.py
    instrumentors.py

tests/
  pixie/
    instrumentation/
      test_spans.py
      test_queue.py
      test_processor.py
```

### 3. Test Coverage Requirements

- **All new code must have tests**: Functions, classes, methods, utilities
- **Bug fixes must include regression tests**: Add a test that would have caught the bug
- **Refactoring must maintain tests**: Verify all existing tests still pass

**What to test:**

- Function inputs and outputs
- Edge cases and boundary conditions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yiouli/pixie-qa](https://github.com/yiouli/pixie-qa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
