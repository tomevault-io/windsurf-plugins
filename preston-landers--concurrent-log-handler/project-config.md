---
trigger: always_on
description: This document provides guidance for AI assistants including Claude Code to
---

# CLAUDE.md

This document provides guidance for AI assistants including Claude Code to
ensure contributions to this repository are consistent, high-quality, and align
with the project's design principles.

## 1. Project Philosophy

The core mission of Concurrent Log Handler (CLH) is to **preserve log records
reliably** in multi-process or even multi-host environments (those with shared
filesystems).

- **Reliability over Performance:** Prioritize data integrity and robust file
  locking. Performance optimizations should never come at the cost of losing
  logs, unless explicitly configurable by the user.
- **Simplicity and Compatibility:** Adhere to the standard Python
  `logging.Handler` interface. New features should feel like natural extensions,
  not radical departures.
- **Clarity:** Concurrency logic is complex. Prioritize clear, well-commented
  code over overly clever or obscure implementations.
- **Error handling:** Fail loudly in development, gracefully in production.
- **Backwards Compatibility:** Changes should not break existing user
  configurations without a clear deprecation path and warning.

## 2. Development Environment Setup

Set up an editable installation with all development dependencies:

```bash
# Assuming Python virtual environment already established
pip install -e .[dev]
```

This installs `hatch`, `pytest`, `black`, `ruff`, `mypy`, and other development
tools specified in `pyproject.toml`.

## 3. Key Development Commands

### Running Tests

The test suite is the primary tool for verifying correctness, especially across
multiple processes.

```bash
# Run tests on the current Python version
pytest

# Run tests with verbose output
pytest -v

# Run a specific test file
pytest tests/test_shutdown_handling.py

# Run a specific test function
pytest tests/test_shutdown_handling.py::test_specific_function -v

# Run tests with coverage report
pytest --cov=src/concurrent_log_handler --cov-report=term-missing

# Generate HTML coverage report
pytest --cov --cov-report=html --cov-report=xml --cov-report=lcov --cov-report=term-missing

# Using hatch (alternative)
hatch test
```

Tests across multiple Python versions are performed in GitHub Actions. Python
3.6+ is supported, though 3.6 and 3.7 are EOL and not in the GitHub Actions
matrix.

### Code Quality (Linting & Formatting)

Code must be formatted and linted before it is considered complete.

```bash
# Auto-format all code
black .

# Check for linting errors and style issues (with auto-fix)
ruff check . --fix

# Perform static type checking
mypy --install-types --non-interactive src/concurrent_log_handler

# On Linux/Unix, run all linting tools at once
./lint.sh
```

### Building the Package

```bash
# Create the source and wheel distributions
hatch build --clean
```

The distributable files will be in the `dist/` folder.

## 4. Core Architecture & Constraints

### File Structure

- **`src/concurrent_log_handler/__init__.py`**: Main module containing all
  handler classes (~1400 lines)
- **`src/concurrent_log_handler/__version__.py`**: Version string
- **`tests/`**: Test suite with multi-process stress tests
- **`docs/`**: Additional documentation, including performance patterns

### Key Components

- **`ConcurrentRotatingFileHandler`**: The base class for size-based log
  rotation. Extends `BaseRotatingHandler` from Python's standard library.
- **`ConcurrentTimedRotatingFileHandler`**: A subclass for time-based rotation
  (extends `TimedRotatingFileHandler`). Can also rotate based on size if
  `maxBytes` is set.
- **`portalocker`**: External library used for all advisory file locking, which
  is the foundation of multi-process safety. This is the only required external
  dependency.

### Architectural Constraints (Critical Rules)

These are non-negotiable constraints that all changes must respect:

1. **Handlers are Not Shared Across Processes:**
   - Each process _must_ create its own instance of the handler.
   - Handlers cannot be serialized or passed between processes (e.g., via
     `multiprocessing` in spawn mode).
   - This is because file lock objects and internal states cannot be safely
     shared across process boundaries.
   - This does **not** apply to threads within a single process; threads can
     share a handler instance.

2. **File-Based Locking:**
   - All inter-process coordination uses a `.lock` file on the filesystem.
   - Lock file is typically `.<logfilename>.lock` in the same directory as the
     log file.
   - The `lock_file_directory` option allows placing lock files elsewhere
     (useful for network filesystems).

3. **Lock Acquisition Per Write:**
   - A file lock is acquired and released for every log message written to disk.
   - This ensures thread-safety and multi-process safety, but has performance
     implications.
   - The `keep_file_open` option (default: `True`) improves performance by
     keeping files open between writes.

4. **Consistent Configuration Required:**
   - All processes writing to the same log file must use identical settings
     (`maxBytes`, `backupCount`, rotation interval, etc.).
   - Mixing CLH handlers with standard library handlers (like
     `RotatingFileHandler`) on the same file leads to unpredictable behavior.

5. **Platform Support:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Preston-Landers/concurrent-log-handler](https://github.com/Preston-Landers/concurrent-log-handler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
