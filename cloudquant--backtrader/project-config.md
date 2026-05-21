---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Backtrader is a Python-based quantitative trading and backtesting framework for mid-to-low frequency strategies. This is a fork that removes metaclass-based metaprogramming in favor of explicit initialization patterns while maintaining API compatibility with the original backtrader.

- *Branch Context**:
- `dev` branch: Active development with 45% performance improvement, tick-level testing, C++ integration
- `master` branch: Stable version aligned with official backtrader
- `remove-metaprogramming`: Legacy branch for metaclass removal (mostly merged into dev)

- *Performance**: The dev branch achieves 45% faster execution through elimination of metaclasses, broker optimization, and Cython-accelerated calculations.

## Development Commands

### Installation

```bash

# Install dependencies

pip install -r requirements.txt

# Compile Cython files (Unix/Mac)

cd backtrader && python -W ignore compile_cython_numba_files.py && cd .. && pip install -U .

# Compile Cython files (Windows)

cd backtrader; python -W ignore compile_cython_numba_files.py; cd ..; pip install -U .

```bash

### Testing

```bash

# Run all tests (parallel execution recommended)

pytest tests/ -n 4 -v

# Run only original tests (excluding crypto tests)

pytest tests/original_tests/ -v

# Run indicator tests

pytest tests/add_tests/test_ind*.py tests/original_tests/test_ind*.py -v

# Run strategy tests

pytest tests/add_tests/test_strategy*.py tests/original_tests/test_strategy*.py -v

# Run analyzer tests

pytest tests/add_tests/test_analyzer*.py tests/original_tests/test_analyzer*.py -v

# Run single test with detailed output

pytest tests/path/to/test_file.py::test_function_name -v --tb=short

# Run tests with coverage

make test-coverage

# Run benchmarks

make benchmark

```bash

### Code Quality

```bash

# Format code (Black)

make format

# Check formatting

make format-check

# Run linter

make lint

# Type checking

make type-check

# Security checks

make security

# Run all quality checks

make quality-check

# Full code optimization (pyupgrade, isort, black, ruff, then test)

bash scripts/optimize_code.sh

```bash

### Documentation

```bash

# Generate all documentation (en + zh)

make docs

# Generate English documentation

make docs-en

# Generate Chinese documentation

make docs-zh

# Build English docs with live reload (for development)

make docs-live

# Open documentation in browser

make docs-view

```bash

### Development Utilities

```bash

# See all available commands

make help

# Clean build artifacts

make clean

# Setup git hooks for development

make git-setup

# Analyze metaclass usage in codebase

make analyze-metaprogramming

```bash

## Architecture Overview

### Core Class Hierarchy (After Metaclass Removal)

The codebase is transitioning from metaclass-based to mixin-based architecture:

1. **Base Layer**: `metabase.py`
   - `BaseMixin`: Provides `donew()` pattern for explicit initialization
   - `findowner()`: Locates owner objects in the call stack
   - Replaces metaclass logic with explicit method calls

1. **Line System**(bottom-up):
   - `LineRoot` → `LineBuffer` → `LineSeries` → `LineIterator`
   - `LineRoot`: Base interface for line operations and period management
   - `LineBuffer`: Data storage with circular buffer support (`linebuffer.py:~1950 lines`)
   - `LineSeries`: Time series operations and data access
   - `LineIterator`: Iteration logic, prenext/next/once phase management

3.**Operational Classes**:

   - `Indicator` (lineiterator.py): Technical indicators base class
   - `Observer`: Chart observers (volume, cash, etc.)
   - `Analyzer`: Performance metrics and statistics
   - `Strategy`: Trading strategy base class
   - `Data/Feed`: Data source management

1. **Engine**: `cerebro.py` (~88K lines)
   - Main orchestration engine
   - Manages strategies, data feeds, brokers, analyzers
   - Handles backtesting execution flow
   - Coordinates the entire backtesting lifecycle

### Critical Initialization Pattern

- *After metaclass removal, initialization follows this pattern:**

```python

# Old metaclass way (deprecated):

# __new__ + metaclass magic

# New explicit way:

def __new__(cls, *args, **kwargs):
    _obj, args, kwargs = cls.donew(*args, **kwargs)
    return _obj

def __init__(self, *args, **kwargs):

# Initialize attributes early

# Call parent class __init__
    super().__init__(*args, **kwargs)

```bash

- *Key Points**:
- `donew()` method replaces metaclass `__call__`
- Owner finding happens in `donew()` via `metabase.findowner()`
- Parameters are initialized before `__init__` is called
- Line buffers are created during parent `__init__` chain

### Indicator Registration System

- *Critical**: Indicators must register themselves with their owner's `_lineiterators` list.

- *Location**: `lineiterator.py:528-556`

- *How it works**:
1. Indicator sets `_ltype = LineIterator.IndType` (value: 0)
2. During `__init__`, if indicator has an owner, it auto-registers
3. Owner's `_next()` method iterates `_lineiterators` to update all indicators

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cloudQuant/backtrader](https://github.com/cloudQuant/backtrader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
