---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Fast LiteLLM is a high-performance Rust acceleration layer for LiteLLM that provides drop-in replacements for performance-critical components. It uses PyO3 to create Python extensions from Rust code, achieving 2-20x performance improvements while maintaining full compatibility with the existing Python API.

## Build Commands

### Initial Setup
```bash
# Install uv (fast Python package manager)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Create virtual environment and install dependencies
uv venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install maturin and development dependencies
uv add --dev maturin

# Development build (creates editable install)
uv run maturin develop

# Or use the automated setup script
./scripts/setup_dev.sh
```

### Building
```bash
# Development build (fast, with debug symbols)
uv run maturin develop

# Release build (optimized)
uv run maturin build --release

# Build wheel for distribution
uv run maturin build --release --out dist

# Build for specific Python version
uv run maturin build --interpreter python3.11
```

### Testing
```bash
# Run all tests
uv run pytest tests/

# Run specific test file
uv run pytest tests/test_accelerator.py

# Run single test
uv run pytest tests/test_basic.py::test_import_package

# Run tests with coverage
uv run pytest tests/ --cov=fast_litellm --cov-report=html

# Run tests excluding slow benchmarks
uv run pytest tests/ -m "not slow"

# Run only integration tests
uv run pytest tests/ -m integration

# Run Rust tests
cargo test

# Run Rust tests with specific module
cargo test --lib core
```

### Code Quality
```bash
# Python formatting
uv run black fast_litellm/ tests/

# Python import sorting
uv run isort fast_litellm/ tests/

# Python linting
uv run flake8 fast_litellm/
uv run mypy fast_litellm/
uv run ruff check fast_litellm/

# Rust formatting
cargo fmt

# Rust linting
cargo clippy -- -D warnings

# Run all quality checks
uv run black --check . && uv run isort --check-only . && uv run flake8 . && uv run mypy fast_litellm/ && cargo fmt -- --check && cargo clippy
```

## Architecture

### Dual-Layer Design

The project has two interconnected layers:

1. **Rust Layer (`src/`)**: High-performance implementations using PyO3
   - `lib.rs`: Main PyO3 module that exports Python functions
   - `core.rs`: Advanced routing with DashMap for lock-free concurrency
   - `tokens.rs`: Token counting (placeholder for tiktoken-rs integration)
   - `connection_pool.rs`: Connection management with atomic operations
   - `rate_limiter.rs`: Token bucket and sliding window rate limiting
   - `feature_flags.rs`: Gradual rollout and canary deployment system
   - `performance_monitor.rs`: Real-time metrics collection

2. **Python Layer (`fast_litellm/`)**: Integration and fallback logic
   - `__init__.py`: Package entry with automatic Rust import and fallback
   - `enhanced_monkeypatch.py`: Smart patching with performance monitoring
   - `feature_flags.py`: Python-side feature flag management
   - `performance_monitor.py`: Performance tracking and comparison
   - Falls back to Python implementations if Rust extensions unavailable

### Monkeypatching Strategy

The system uses enhanced monkeypatching to replace LiteLLM components:

1. Import detection: When `fast_litellm` is imported, it automatically attempts to load Rust extensions
2. Smart replacement: `PerformanceWrapper` class wraps functions with:
   - Feature flag checking (per-request rollout control)
   - Performance monitoring (timing and metrics)
   - Automatic fallback on errors
3. Gradual rollout: Feature flags control which requests use Rust vs Python

### Key Design Patterns

1. **Lock-Free Concurrency**: Uses DashMap instead of standard HashMap for thread-safe operations without locks
2. **Atomic Operations**: Rate limiters and connection pools use atomic integers for concurrent access
3. **Zero-Copy Returns**: PyO3 conversions minimize data copying between Rust and Python
4. **Feature Flags**: Every Rust component can be individually enabled/disabled with percentage-based rollout

## Critical Files

### Build Configuration
- `pyproject.toml`: Main package configuration with maturin settings
- `Cargo.toml`: Rust package configuration with PyO3 dependencies
- `[tool.maturin]` section: Configures module name as `fast_litellm._rust`

### Entry Points
- `fast_litellm/__init__.py`: Main package entry, imports from `._rust` module
- `src/lib.rs`: Rust entry point with `#[pymodule]` definition

### Integration Points
- The Rust module is compiled as `_rust.so` and imported as `fast_litellm._rust`
- All Rust functions are exposed via `#[pyfunction]` decorators
- JSON data is converted between `serde_json::Value` and Python objects

## Development Workflow

### Adding New Rust Functionality

1. Add Rust implementation in appropriate `src/*.rs` file
2. Export function in `src/lib.rs` with `#[pyfunction]` decorator
3. Add to module in `lib.rs` `#[pymodule]` function
4. Rebuild with `uv run maturin develop`
5. Add Python wrapper in `fast_litellm/` if needed
6. Add tests in `tests/`

### Testing Cycle


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neul-labs/fast-litellm](https://github.com/neul-labs/fast-litellm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
