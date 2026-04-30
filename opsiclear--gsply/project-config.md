---
trigger: always_on
description: This file provides context and instructions for AI coding agents working on the gsply project.
---

# AGENTS.md

This file provides context and instructions for AI coding agents working on the gsply project.

## Project Overview

**gsply** is an ultra-fast Gaussian Splatting PLY I/O library for Python.

- **Language**: Pure Python (3.10+)
- **Core Dependencies**: NumPy, Numba (JIT acceleration)
- **Optional Dependencies**: PyTorch (GPU acceleration via GSTensor)
- **Performance**: 93M Gaussians/sec read, 57M Gaussians/sec write
- **Key Features**: Zero-copy optimization, compressed format support, GPU integration, SOG format support
- **Current Version**: 0.2.12

## Development Environment Setup

### Installation

```bash
# Clone repository
git clone https://github.com/OpsiClear/gsply.git
cd gsply

# Install in editable mode with dev dependencies
pip install -e ".[dev]"

# Optional: Install PyTorch for GPU features (testing GSTensor)
pip install torch torchvision --index-url https://download.pytorch.org/whl/cpu
```

### Using uv (preferred by maintainer)

Per CLAUDE.md instructions, the maintainer prefers `uv` for running scripts:

```bash
# Run tests
uv run pytest

# Run scripts
uv run python script.py

# Install pre-commit (for git hooks)
uv pip install pre-commit
pre-commit install
```

### Pre-commit Hooks

The project uses pre-commit to ensure code quality before commits:

```bash
# Install hooks (one-time setup)
pre-commit install

# Run all hooks manually
pre-commit run --all-files

# Run specific hook
pre-commit run ruff --all-files

# Run mypy manually (not run automatically due to pre-existing issues)
pre-commit run --hook-stage manual mypy --all-files

# Update hook versions
pre-commit autoupdate
```

**Hooks configured:**
- Trailing whitespace removal
- End of file fixing
- YAML/TOML validation
- Large file detection
- Merge conflict detection
- Line ending normalization (LF)
- Ruff linter and formatter
- MyPy type checking (manual stage)

### Project Structure

```
gsply/
├── src/gsply/           # Main package
│   ├── __init__.py      # Public API exports
│   ├── reader.py        # PLY reading (plyread, decompress)
│   ├── writer.py        # PLY writing (plywrite, compress)
│   ├── gsdata.py        # GSData dataclass (CPU container)
│   ├── formats.py       # Format detection and constants
│   ├── utils.py         # Utility functions (sh2rgb, rgb2sh, logit, sigmoid, apply_pre_activations, apply_pre_deactivations)
│   ├── sog_reader.py    # SOG format reading (sogread, optional dependency)
│   ├── py.typed         # Type checking marker (PEP 561)
│   └── torch/           # Optional PyTorch integration
│       ├── __init__.py  # Conditional import (checks torch availability)
│       ├── gstensor.py  # GSTensor GPU dataclass
│       ├── compression.py  # GPU compression/decompression
│       └── io.py        # GPU I/O (plyread_gpu, plywrite_gpu)
├── tests/               # Test suite (365 tests)
├── benchmarks/          # Performance benchmarks
├── docs/                # Documentation
└── .github/workflows/   # CI/CD pipelines
```

## Testing

### Running Tests

```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=gsply --cov-report=html

# Run specific test file
pytest tests/test_api.py

# Run tests matching pattern
pytest -k "test_plyread"

# Run PyTorch tests only (requires torch installed)
pytest -k "torch or gstensor"
```

### Test Data Generation

Tests automatically generate synthetic data. Some tests use real PLY files:
- Set `GSPLY_TEST_DATA_DIR` environment variable to point to real PLY files (optional)
- Tests will create `export_with_edits/` directory with synthetic data

### CI/CD

- **Location**: `.github/workflows/test.yml`
- **Matrix Testing**: Ubuntu, Windows, macOS × Python 3.10, 3.11, 3.12, 3.13
- **Separate Torch Tests**: `.github/workflows/test.yml` has dedicated `test-torch` job
- **Benchmarks**: `.github/workflows/benchmark.yml` runs performance tests
- **All tests must pass** before merging

### Test Count

Current test count: **365 tests** (documented in README.md)
- Update this count in README if adding/removing tests
- Includes 17 new tests for fused activation kernels (tests/test_pre_activations.py)

## Code Style and Conventions

### Type Hints

- **Required**: Use Python 3.12+ type hint syntax
- **Tool**: `tyro` for type management and CLI (per CLAUDE.md)
- **Example**: `list[int]` not `List[int]`, `dict[str, Any]` not `Dict[str, Any]`

### CLI Argument Parsing

- **Use argparse** for this project (per CLAUDE.md)
- Not tyro for CLI despite being preferred for type management

### Logging

- **Use logging module** instead of print statements for info (per CLAUDE.md)
- Example: `logging.info()` not `print()`

### Unicode and Emojis

- **Never use emojis or Unicode characters** in code or output (per CLAUDE.md)
- Use ASCII only: `[OK]`, `[FAIL]`, `->` instead of ✓, ✅, ❌, →

### Code Formatting

- **Linter**: Ruff (`ruff check .`)
- **Type Checker**: MyPy (`mypy src/`)
- **Line Length**: 100 characters (conventional Python)
- Follow PEP 8 conventions

### Naming Conventions

- **Variables**: `snake_case`
- **Classes**: `PascalCase` (GSData, GSTensor)
- **Constants**: `UPPER_SNAKE_CASE` (SH_C0)
- **Private**: Prefix with `_` (_base, _recreate_from_base)

### Documentation Style


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpsiClear/gsply](https://github.com/OpsiClear/gsply) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
