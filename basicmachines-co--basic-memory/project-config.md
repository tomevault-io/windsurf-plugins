---
trigger: always_on
description: Basic Memory is a local-first knowledge management system built on the Model Context Protocol (MCP). It enables
---

# AGENTS.md - Basic Memory Project Guide

## Project Overview

Basic Memory is a local-first knowledge management system built on the Model Context Protocol (MCP). It enables
bidirectional communication between LLMs (like Claude) and markdown files, creating a personal knowledge graph that can
be traversed using links between documents.

## CODEBASE DEVELOPMENT

### Project information

See the [README.md](README.md) file for a project overview.

### Build and Test Commands

- Install: `just install` or `pip install -e ".[dev]"`
- Run all tests (SQLite + Postgres): `just test`
- Run all tests against SQLite: `just test-sqlite`
- Run all tests against Postgres: `just test-postgres` (uses testcontainers)
- Run unit tests (SQLite): `just test-unit-sqlite`
- Run unit tests (Postgres): `just test-unit-postgres`
- Run integration tests (SQLite): `just test-int-sqlite`
- Run integration tests (Postgres): `just test-int-postgres`
- Run impacted tests: `just testmon` (pytest-testmon; only tests affected by changed code)
- Run MCP smoke test: `just test-smoke`
- Fast local loop: `just fast-check` (default iteration flow)
- Local consistency check: `just doctor`
- Generate HTML coverage: `just coverage`
- Single test: `pytest tests/path/to/test_file.py::test_function_name`
- Run benchmarks: `pytest test-int/test_sync_performance_benchmark.py -v -m "benchmark and not slow"`
- Lint: `just lint` or `ruff check . --fix`
- Type check: `just typecheck` or `uv run ty check src tests test-int`
- Type check (pyright): `just typecheck-pyright` or `uv run pyright`
- Format: `just format` or `uv run ruff format .`
- Run all code checks: `just check` (runs lint, format, typecheck, test)
- Create db migration: `just migration "Your migration message"`
- Run development MCP Inspector: `just run-inspector`

**Note:** Project requires Python 3.12+ (uses type parameter syntax and `type` aliases introduced in 3.12)

**Postgres Testing:** Uses [testcontainers](https://testcontainers-python.readthedocs.io/) which automatically spins up a Postgres instance in Docker. No manual database setup required - just have Docker running.

**Doctor Note:** `just doctor` runs with a temporary HOME/config so it won't touch your local Basic Memory settings. It leaves temp dirs in `/tmp` (safe to ignore or remove).

**Testmon Note:** When no files have changed, `just testmon` may collect 0 tests. That's expected and means no impacted tests were detected.

### Code/Test/Verify Loop (fast path)

1) **Code:** make changes.
2) **Test:** `just fast-check` (lint/format/typecheck + pytest-testmon impacted tests for changed code).
3) **Verify:** `just doctor` (end-to-end file ↔ DB loop in a temp project).
4) **Full gate (when needed):** `just test` or `just check` for SQLite + Postgres.

Run `just test-smoke` when you specifically need the MCP smoke flow.

If testmon is “cold,” the first run may be long. Subsequent runs get much faster.

### Test Structure

- `tests/` - Unit tests for individual components (mocked, fast)
- `test-int/` - Integration tests for real-world scenarios (no mocks, realistic)
- Both directories are covered by unified coverage reporting
- Benchmark tests in `test-int/` are marked with `@pytest.mark.benchmark`
- Slow tests are marked with `@pytest.mark.slow`
- Smoke tests are marked with `@pytest.mark.smoke`

### Code Style Guidelines

- Line length: 100 characters max
- Python 3.12+ with full type annotations (uses type parameters and type aliases)
- Format with ruff (consistent styling)
- Import order: standard lib, third-party, local imports
- Naming: snake_case for functions/variables, PascalCase for classes
- Prefer async patterns with SQLAlchemy 2.0
- Use Pydantic v2 for data validation and schemas
- CLI uses Typer for command structure
- API uses FastAPI for endpoints
- Follow the repository pattern for data access
- Tools communicate to api routers via the httpx ASGI client (in process)

### Code Change Guidelines

- **Full file read before edits**: Before editing any file, read it in full first to ensure complete context; partial reads lead to corrupted edits
- **Minimize diffs**: Prefer the smallest change that satisfies the request. Avoid unrelated refactors or style rewrites unless necessary for correctness
- **No speculative getattr**: Never use `getattr(obj, "attr", default)` when unsure about attribute names. Check the class definition or source code first
- **Fail fast**: Write code with fail-fast logic by default. Do not swallow exceptions with errors or warnings
- **No fallback logic**: Do not add fallback logic unless explicitly told to and agreed with the user
- **No guessing**: Do not say "The issue is..." before you actually know what the issue is. Investigate first.

### Literate Programming Style

Code should tell a story. Comments must explain the "why" and narrative flow, not just the "what".

**Section Headers:**
For files with multiple phases of logic, add section headers so the control flow reads like chapters:
```python
# --- Authentication ---
# ... auth logic ...

# --- Data Validation ---
# ... validation logic ...

# --- Business Logic ---
# ... core logic ...
```

**Decision Point Comments:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [basicmachines-co/basic-memory](https://github.com/basicmachines-co/basic-memory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
