---
trigger: always_on
description: This file provides guidance for Claude Code when working with the rustest codebase.
---

# CLAUDE.md

This file provides guidance for Claude Code when working with the rustest codebase.

## Project Overview

**rustest** is a Rust-powered pytest-compatible test runner focused on raw performance. It delivers massive speedups (8.5x average, up to 19x faster) while maintaining familiar pytest ergonomics.

- **Languages**: Rust (core engine) + Python (user API/CLI)
- **Build System**: Maturin (PyO3 bridge for Rust-Python integration)
- **Python Support**: 3.10 - 3.14
- **License**: MIT

## Project Structure

```
src/                          # Rust core (rustest-core crate)
├── lib.rs                    # Main entry point, PyO3 module
├── discovery.rs              # Fast test file discovery
├── execution.rs              # Test execution engine
├── model.rs                  # Data structures (TestCase, Fixture, etc.)
├── python_support.rs         # Rust-Python bridge
├── mark_expr.rs              # Mark expression parsing
├── cache.rs                  # Caching logic
└── output/                   # Output formatting

python/rustest/               # Python package (user API)
├── __init__.py               # Public API exports
├── __main__.py               # CLI entry point
├── decorators.py             # @fixture, @parametrize, @mark, etc.
├── builtin_fixtures.py       # tmp_path, tmpdir, monkeypatch, capsys, capfd, request
├── cli.py                    # Command-line interface
├── core.py                   # Wrapper around Rust layer
├── approx.py                 # Numeric comparison helper
└── compat/pytest.py          # pytest compatibility layer

python/tests/                 # Python unit tests
tests/                        # Integration test suite
examples/tests/               # Example test suite
docs/                         # MkDocs documentation
```

## Development Commands

### Initial Setup
```bash
uv sync --all-extras          # Install dependencies
uv run maturin develop        # Build Rust extension
```

### Building
```bash
uv run maturin develop        # Rebuild Rust extension after changes
poe dev                       # Alias for above
poe build                     # Build package for distribution
```

### Testing
```bash
# Python unit tests
uv run poe pytests
uv run pytest python/tests -v

# Integration tests
uv run pytest tests/ examples/tests/ -v

# Run with rustest itself
uv run python -m rustest tests/ examples/tests/ -v

# Rust tests
cargo test

# Example tests
uv run rustest examples/tests/
```

### Formatting (REQUIRED before commits)
```bash
# Rust - ALWAYS run for Rust changes
cargo fmt
cargo fmt --check             # Verify formatting

# Python - ALWAYS run for Python changes
uv run ruff format python
uv run ruff format --check python
```

### Linting
```bash
# Rust
cargo clippy --lib -- -D warnings

# Python
uv run ruff check python
uv run basedpyright python    # Type checking
```

### Pre-commit (runs all checks)
```bash
uv run pre-commit install     # One-time setup
uv run pre-commit run --all-files
```

### Task Runner Shortcuts
```bash
poe dev       # Rebuild Rust extension
poe pytests   # Run Python tests
poe lint      # Check Python style
poe typecheck # Type check Python
poe fmt       # Format Rust
poe tests     # Run integration and example tests
```

## Code Style and Conventions

### Rust
- Follow standard Rust conventions (rustfmt)
- Use `cargo clippy` with `-D warnings` (treat warnings as errors)
- Document public APIs with doc comments
- Use `rayon` for parallelization where appropriate

### Python
- Follow Ruff formatting and linting rules
- Use type hints everywhere (checked by basedpyright)
- Public API is exported from `python/rustest/__init__.py`
- Decorators go in `decorators.py`, fixtures in `builtin_fixtures.py`

## Architecture Notes

### Hybrid Design
1. **Rust Core** (`src/`) - High-performance engine for:
   - Test discovery (globset, regex)
   - Test execution (rayon for parallelization)
   - Result formatting

2. **Python Layer** (`python/rustest/`) - User-friendly API for:
   - Decorators (`@fixture`, `@parametrize`, `@mark`)
   - Built-in fixtures
   - CLI interface
   - pytest compatibility

3. **PyO3/Maturin Bridge** - Compiled Rust exposed as `rustest.rust` module

### Key Entry Points
- CLI: `python -m rustest` → `__main__.py` → `cli.py:main()`
- Python API: `from rustest import fixture, mark, parametrize`
- Test Discovery: `src/discovery.rs:discover_tests()`
- Test Execution: `src/execution.rs:run_collected_tests()`

## Pre-commit Requirements

**CRITICAL**: Before any commit, ALL of the following must pass. CI will fail otherwise:

### Rust Changes
- `cargo fmt` - Format code
- `cargo fmt --check` - Verify formatting passes
- `cargo clippy --lib -- -D warnings` - No lint warnings
- `cargo build` - Must compile without errors

### Python Changes
- `uv run ruff format python` - Format code
- `uv run ruff check python` - No lint errors
- `uv run basedpyright python` - **ALL type checks must pass**

### All Changes
- `uv run pre-commit run --all-files` - Run complete check suite

### Tests (ALL must pass)
- `cargo test` - Rust unit tests
- `uv run pytest python/tests -v` - Python unit tests (via pytest)
- `uv run pytest tests/ examples/tests/ -v` - Integration tests (via pytest)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Apex-Engineers-Inc/rustest](https://github.com/Apex-Engineers-Inc/rustest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
