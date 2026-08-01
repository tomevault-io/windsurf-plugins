---
trigger: always_on
description: **Wily** is a Python CLI tool for tracking, reporting on, and graphing code complexity metrics across git history. It analyzes Python source code using operators (cyclomatic complexity, Halstead metrics, maintainability index, raw line counts) and stores results in a Parquet-based cache.
---

# AGENTS.md

## Project Overview

**Wily** is a Python CLI tool for tracking, reporting on, and graphing code complexity metrics across git history. It analyzes Python source code using operators (cyclomatic complexity, Halstead metrics, maintainability index, raw line counts) and stores results in a Parquet-based cache.

**Architecture**: Hybrid Python + Rust (PyO3/maturin). The performance-critical backend (parsing, metric computation, Parquet storage, git operations) is written in Rust and exposed to Python as a native extension module (`wily.backend`). The CLI, configuration, command orchestration, and output formatting are in Python.

**Current branch**: `v2` (major rewrite from v1 — uses Rust backend + Parquet storage instead of pure Python + JSON).

## Commands

### Setup & Build

```bash
# Install for development (compiles Rust backend + installs Python package)
uv sync --all-groups
maturin develop

# Build release wheel
maturin build --release
```

### Testing

```bash
# Run all tests (unit + integration) with coverage
uv run pytest

# Run a specific test file
uv run pytest test/unit/test_operators.py

# Run a specific test
uv run pytest test/integration/test_build.py::test_build -v
```

- Test framework: **pytest ~7.2** with `pytest-cov`
- Test directory: `test/` (configured in `pyproject.toml` under `[tool.pytest.ini_options]`)
- Tests use `click.testing.CliRunner` for CLI integration tests
- Fixtures in `test/conftest.py` provide `gitdir` (git repo with 3 commits), `builddir` (gitdir + wily cache), and ipynb variants
- The `cache_path` fixture (autouse) creates a temp HOME to isolate wily cache per test
- Coverage config in `setup.cfg` under `[coverage:run]`

### Linting

```bash
# Python linting
uv tool run ruff check .

# Rust linting
cargo clippy --manifest-path backend/Cargo.toml -- -D warnings

# Rust formatting check
cargo fmt --manifest-path backend/Cargo.toml --check
```

- Python linter: **ruff ~0.14** (configured in `pyproject.toml` under `[tool.ruff]`)
- Line length: **217** (very permissive)
- Ruff rules enabled: B, C4, C9, D (docstrings), E, F, I (isort), PL, S (security), U, W, YTT
- Test files exempt from docstring rules (`D`) and `S101` (assert)
- Max McCabe complexity: 24, max branches: 35

### i18n / Localization

```bash
# Extract translatable strings
make extract_messages

# Compile message catalogs
make compile_messages
```

## Code Organization

```
src/wily/                   # Python package root (specified by tool.maturin python-source)
├── __init__.py             # Version, logger setup, format helpers
├── __main__.py             # Click CLI entry point (all subcommands defined here)
├── backend.pyi             # Type stubs for the Rust extension module
├── backend.*.pyd           # Compiled Rust extension (platform-specific)
├── cache.py                # Cache directory operations (create, clean, exists, list)
├── defaults.py             # Default constants (archiver, config path, max revisions)
├── lang.py                 # i18n/gettext support via `_()` function
├── operators.py            # Metric/Operator definitions and resolution functions
├── config/
│   ├── __init__.py         # Config loading from wily.cfg files
│   └── types.py            # WilyConfig dataclass
├── commands/
│   ├── build.py            # Build/index git history into Parquet cache
│   ├── diff.py             # Compare uncommitted files against cached metrics
│   ├── graph.py            # Generate Plotly HTML graphs of metrics over time
│   ├── index.py            # Show cache revision history
│   ├── list_metrics.py     # List available metrics
│   ├── rank.py             # Rank files by a metric
│   └── report.py           # Show metric history for a specific file
├── archivers/
│   ├── __init__.py         # BaseArchiver, Archiver, Revision types
│   ├── git.py              # Git archiver (uses Rust backend for git ops)
│   └── filesystem.py       # Filesystem archiver (non-git)
├── helper/
│   ├── __init__.py         # print_table (Rich), generate_cache_path
│   └── custom_enums.py     # ReportFormat enum
├── templates/              # HTML report templates
└── locales/                # gettext translation files (en, en_AU, de, ja)

backend/                    # Rust crate (PyO3 extension module)
├── Cargo.toml              # Rust dependencies (pyo3, ruff_python_parser, arrow, parquet, git2)
├── src/
│   ├── lib.rs              # PyO3 module registration
│   ├── storage.rs          # WilyIndex: Parquet read/write, analyze_revision, analyze_files
│   ├── git.rs              # Git operations (get_revisions, checkout, find_revision)
│   ├── files.rs            # File iteration (iter_filenames)
│   ├── raw.rs              # Raw line count metrics
│   ├── cyclomatic.rs       # Cyclomatic complexity computation
│   ├── halstead.rs         # Halstead metrics computation
│   └── maintainability.rs  # Maintainability index computation
└── benches/                # Criterion benchmarks
    └── analyze_revision/

test/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tonybaloney/wily](https://github.com/tonybaloney/wily) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
