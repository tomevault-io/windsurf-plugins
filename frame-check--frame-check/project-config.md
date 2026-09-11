---
trigger: always_on
description: **frame-check** is a static analyzer for pandas DataFrames that catches column access errors before runtime. Think of it as `mypy` for DataFrame columns - it tracks DataFrame schemas through code and reports errors for missing columns.
---

# AGENTS.md - Guide for AI Agents

## Project Overview

**frame-check** is a static analyzer for pandas DataFrames that catches column access errors before runtime. Think of it as `mypy` for DataFrame columns - it tracks DataFrame schemas through code and reports errors for missing columns.

- **Repository**: https://github.com/frame-check/frame-check
- **Python version**: >=3.12
- **Package manager**: [uv](https://docs.astral.sh/uv/)
- **Linter/Formatter**: Ruff
- **Type checker**: mypy (basic mode)

## Repository Structure

```
frame-check/
├── conftest.py                # Pytest configuration (at root for --support to work)
├── frame-check-core/          # Core type checker package
│   ├── src/frame_check_core/
│   │   ├── checker.py         # Main AST visitor (entry point)
│   │   ├── tracker.py         # Column dependency tracking
│   │   ├── refs.py            # Type guards and ColumnRef dataclass
│   │   ├── handlers/          # Operation handlers (column state changes)
│   │   │   ├── models.py      # PD/DF registries for operation handlers
│   │   │   ├── pandas.py      # pd.* function handlers
│   │   │   └── dataframe.py   # df.* method handlers
│   │   ├── extractors/        # Column extractors (column references)
│   │   │   ├── registry.py    # Extractor registry
│   │   │   ├── column.py      # df['col'] patterns
│   │   │   └── binop.py       # df['A'] + df['B'] patterns
│   │   ├── diagnostic/        # Error message generation
│   │   ├── config/            # Configuration management
│   │   ├── models/            # Data models
│   │   ├── ast/               # AST utilities
│   │   └── util/              # Utility functions
│   └── tests/                 # Test suite
│       ├── features/           # Feature/API completeness tests (use @pytest.mark.support)
│       ├── extractors/         # Extractor tests
│       ├── config/             # Config tests
│       ├── diagnostic/         # Diagnostic tests
│       └── util/               # Utility tests
├── frame-check-lsp/           # Language Server Protocol implementation
│   └── src/frame_check_lsp/   # LSP server code
├── frame-check-extensions/     # Editor extensions
│   ├── vscode/                # VS Code extension (TypeScript)
│   └── zed/                   # Zed editor extension (Rust)
├── scripts/                   # Scripts including features.toml
├── docs/                      # MkDocs documentation
├── pyproject.toml             # Workspace root config
├── justfile                    # Just commands (test, docs, mypy)
└── zensical.toml              # Docs configuration
```

## Development Setup

```bash
# Clone and setup
git clone https://github.com/frame-check/frame-check.git
cd frame-check

# Create environment and install dependencies
uv sync

# Install pre-commit hooks
uv run pre-commit install
```

## Common Commands

```bash
# Run tests
uv run pytest

# Run tests with verbosity
uv run pytest -v

# Run a specific test file
uv run pytest frame-check-core/tests/test_checker.py

# Run benchmarks
uv run pytest --codspeed

# Type checking
uv run mypy . --check-untyped-defs --exclude scripts

# Linting and formatting (via pre-commit)
uv run ruff check --output-format=github .
uv run ruff format --diff

# Or use just commands
just test          # alias: just t
just docs          # alias: just d
just mypy          # alias: just m
just docs build    # Build documentation
```

## Architecture

### Core Concepts

frame-check works by:
1. **Parsing** Python AST to find DataFrame operations
2. **Tracking** column state changes via handlers
3. **Extracting** column references via extractors
4. **Validating** that accessed columns exist

### Handlers vs Extractors

| Module | Purpose | Example |
|--------|---------|---------|
| **Handlers** | Track column state changes (CREATE/MODIFY) | `pd.DataFrame({'A': [1]})` creates 'A' |
| **Extractors** | Identify column references (READ) | `df['A'] + df['B']` reads 'A' and 'B' |

### Registry Patterns

**Pandas Functions** (`handlers/pandas.py`):
```python
@PD.register("read_excel")
def pd_read_excel(args, keywords) -> PDFuncResult:
    ...
```

**DataFrame Methods** (`handlers/dataframe.py`):
```python
@DF.register("drop")
def df_drop(columns, args, keywords) -> DFFuncResult:
    ...
```

**Extractors** (`extractors/registry.py`):
```python
EXTRACTORS: list[ExtractorFunc] = [
    extract_column_ref,
    extract_column_refs_from_binop,
    # Add new extractors here
]
```

## Adding Features

### 1. Adding a Pandas Function Handler

Location: `frame-check-core/src/frame_check_core/handlers/pandas.py`

```python
@PD.register("read_excel")
def pd_read_excel(args: list[ast.expr], keywords: list[tuple[str, ast.expr]]) -> PDFuncResult:
    # Return columns that this function creates
    return PDFuncResult(columns={"col1", "col2"})
```

### 2. Adding a DataFrame Method Handler

Location: `frame-check-core/src/frame_check_core/handlers/dataframe.py`

```python
@DF.register("drop")
def df_drop(columns: ColumnRef, args: list[ast.expr], keywords: list[tuple[str, ast.expr]]) -> DFFuncResult:
    # Return columns to remove

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [frame-check/frame-check](https://github.com/frame-check/frame-check) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
