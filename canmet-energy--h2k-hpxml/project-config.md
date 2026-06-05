---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Context

This is the H2K-HPXML translation tool - a Canadian government project (CMHC/NRCan) that converts Hot2000 (H2K) building energy models to US DOE's HPXML format for EnergyPlus simulation. The goal is to provide sub-hourly energy analysis for Canada's 6,000+ housing archetypes.

**Project Status**: Phase 1 (loads) and Phase 2 (HVAC systems) complete. Phase 3 (multi-unit residential buildings) pending.

**Branch Strategy**:
- `main` - Production branch for pull requests
- Feature branches - For development work

**Python Version**: Requires Python 3.12+

## Essential Commands

### Development Setup

#### Local Development
```bash
# Install in development mode
uv pip install -e .

# Setup configuration and dependencies
os-setup --setup
os-setup --auto-install

# Verify setup
os-setup --check-only
```

#### Docker Development
```bash
# Using DevContainer (recommended for VS Code users)
# Open project in VS Code and select "Reopen in Container"
```

### Testing
```bash
# Run all tests
pytest

# Run specific test types
pytest tests/unit/                    # Unit tests only
pytest tests/integration/             # Integration tests only
pytest --run-baseline                 # Generate baseline data (WARNING: overwrites golden files)

# Run tests in parallel (faster execution)
pytest -n auto                       # Auto-detect number of CPU cores
pytest -n 4                          # Use 4 parallel workers
pytest tests/integration/test_regression.py -n auto  # Parallel regression tests only

# Code coverage
pytest --cov=src/h2k_hpxml          # Run tests with coverage report
pytest --cov=src/h2k_hpxml --cov-report=html  # Generate HTML coverage report
pytest --cov=src/h2k_hpxml --cov-report=term-missing  # Show missing lines in terminal
pytest -n auto --cov=src/h2k_hpxml  # Parallel tests with coverage

# Generate baseline data (alternative method)
uv run python tests/utils/generate_baseline_data.py  # Direct script execution

# Clean up cache files and temporary data (cross-platform)
uv run python tools/cleanup.py              # Removes __pycache__, tool caches, temp files

# Run single test
pytest tests/unit/test_core_translator.py::TestH2KToHPXML::test_valid_translation_modes -v

# Test interactive demo (cross-platform)
python3 tests/utils/demo_test_automation.py           # Cross-platform demo test with scripted input
python3 tests/utils/demo_test_automation.py --cleanup # Clean up demo test files
pytest tests/integration/test_demo.py                 # Pytest integration tests for demo

# Multi-version Python testing with tox
# First-time setup: Install Python versions
uv python install 3.10 3.11 3.12 3.13  # Install all supported Python versions

# Run tox tests
tox                                    # Test against all Python versions (3.10-3.13)
tox -e py310                          # Test with Python 3.10 only
tox -e py311                          # Test with Python 3.11 only
tox -e py312                          # Test with Python 3.12 only
tox -e py313                          # Test with Python 3.13 only
tox -p auto                           # Run all environments in parallel (faster)
tox -e py312 -- -v                    # Pass extra arguments to pytest (e.g., verbose mode)
tox -e py312 -- tests/unit/           # Run specific test directory with tox
```

### Code Quality
```bash
# Install development dependencies first
uv pip install -e ".[dev]"

# Testing
pytest                              # Run all tests
pytest tests/unit/                  # Unit tests only
pytest tests/integration/           # Integration tests only
pytest -n auto                     # Run tests in parallel (faster)
pytest -v                          # Verbose output
pytest -x                          # Stop on first failure

# Code coverage
pytest --cov=src/h2k_hpxml         # Run tests with coverage
pytest --cov=src/h2k_hpxml --cov-report=html  # Generate HTML coverage report (htmlcov/)
pytest --cov=src/h2k_hpxml --cov-report=term-missing  # Show which lines are missing coverage

# Code formatting and linting
black src/ tests/                   # Format code (line length: 100)
ruff check src/ tests/              # Lint code
ruff check --fix src/ tests/        # Auto-fix linting issues
```

### Documentation

#### Building Documentation

```bash
# Install documentation dependencies (included in dev dependencies)
uv pip install -e ".[dev]"

# Build HTML documentation
cd docs && sphinx-build -b html source build

# Live preview with auto-reload (opens browser at http://127.0.0.1:8000)
cd docs && sphinx-autobuild source build

# Check for broken links
cd docs && sphinx-build -b linkcheck source build

# Clean build directory
cd docs && rm -rf build/

# Alternative: Build from project root
sphinx-build -b html docs/source docs/build
```

#### Documentation Structure

```
docs/
├── source/               # Sphinx source files
│   ├── conf.py          # Sphinx configuration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [canmet-energy/h2k-hpxml](https://github.com/canmet-energy/h2k-hpxml) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
