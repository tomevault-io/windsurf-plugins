---
trigger: always_on
description: Enables interacting with remote Python objects as if they were local.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Code Style Preferences

- **Minimum Python version is 3.9**. Use native generic types (`list[str]`, `dict[str, int]`) instead of importing from `typing`. However, `Optional[X]` still requires `from typing import Optional` since `X | None` syntax requires Python 3.10+.

- **Data class libraries: Use `attrs` or `pydantic`, never `dataclasses`.**
  - **pydantic (`BaseModel`)**: Use for configuration parsing, external input validation, and serialization (e.g., parsing `pyproject.toml`, user-provided config). Leverage validators, `model_dump()`, and `Field()` for these use cases.
  - **attrs (`@define`)**: Use for internal data structures that don't need validation. Use `field()` for advanced options like `factory`, `converter`, or `init=False`. Prefer `field(factory=...)` over `Factory(...)`.
  - **Do not use `@dataclass`** from the standard library. Convert any existing `@dataclass` to `@define` from attrs.

  ```python
  # Good - attrs for internal data structures
  from attrs import define, field


  @define
  class Implementation:
      name: str
      version: tuple[int, int, int] = (0, 0, 0)
      arch: Optional[str] = field(default=None, converter=_arch_converter)
      files: list[str] = field(factory=list)
      id: int = field(factory=lambda: next(_counter), init=False)


  # Good - pydantic for config/external input
  from pydantic import BaseModel, field_validator


  class BelayConfig(BaseModel):
      name: Optional[str] = None
      dependencies: dict[str, DependencyList] = {}

      @field_validator("dependencies", mode="before")
      @classmethod
      def preprocess(cls, v):
          return _preprocess(v)


  # Bad - don't use dataclasses
  from dataclasses import dataclass


  @dataclass  # Don't do this
  class MyData:
      value: str
  ```

---

# Belay Architecture Overview

This document describes the high-level architecture of the Belay codebase, which provides a Python framework for controlling MicroPython and CircuitPython devices over various communication channels (serial, telnet, WebREPL, etc.).

## Table of Contents

1. [Common Development Commands](#common-development-commands)
2. [Core Architecture](#core-architecture)
3. [Main Components](#main-components)
4. [Device Decorators & Executors](#device-decorators--executors)
5. [Communication Layer](#communication-layer)
6. [Code Synchronization](#code-synchronization)
7. [Package Manager](#package-manager)
8. [CLI Structure](#cli-structure)
9. [Key Abstractions](#key-abstractions)
10. [Testing](#testing)

---

## Common Development Commands

This project uses Poetry for dependency management.

### Setup

```bash
# Install dependencies
poetry install

# Activate virtual environment
poetry shell
# Or use prefix for each command:
poetry run <command>
```

### Running Tests

```bash
# Run unit tests only (fast)
poetry run python -m pytest tests

# Run with coverage
poetry run python -m pytest --cov=belay --cov-report=term --cov-report=xml tests

# Run unit + integration tests (requires emulated device)
poetry run python -m pytest --network tests tests/integration

# Run a single test file
poetry run python -m pytest tests/test_device.py

# Run a single test
poetry run python -m pytest tests/test_device.py::test_function_name

# Run with verbose output
poetry run python -m pytest tests -v
```

**Note:** Integration tests use rp2040js emulator and are skipped on Windows by default. Tests use the `--max-runs=3` (flaky) and `--timeout=240` flags automatically.

### Linting and Formatting

```bash
# Run pre-commit hooks manually
poetry run pre-commit run --all-files

# Install pre-commit hooks
poetry run pre-commit install

# Format code with black
poetry run black belay

# Lint with ruff
poetry run ruff check belay

# Auto-fix ruff issues
poetry run ruff check --fix belay
```

### Building Documentation

```bash
# Build HTML docs
poetry run sphinx-build -b html docs/source/ docs/build/html
```

### Integration Tests (Docker)

```bash
# Build Docker image for integration testing
make integration-build

# Run integration tests
make integration-test

# Interactive bash in Docker container
make integration-bash
```

---

## Core Architecture

### Design Philosophy

Belay is a remote code execution framework that bridges the Python host (e.g., a laptop) with a MicroPython/CircuitPython device over a serial or network connection. The key architectural principle is that users define functions and methods on the host, which are then executed on the device.

**Main entry point:** `/Users/brianpugh/projects/belay/belay/__init__.py`
- Exports: `Device`, `DeviceMeta`, `ProxyObject`, and various exceptions

### Execution Flow

```
Host Code → Belay Device Object → Executor → Pyboard → Serial/Network → Device REPL → Return Value
```

The flow involves:
1. User defines functions decorated with `@device.task`, `@device.setup`, etc.
2. Belay extracts the source code and sends it to the device
3. Pyboard communicates via serial/network protocols
4. Code executes on-device in MicroPython/CircuitPython REPL
5. Results are returned and parsed on the host

---

## Main Components

### 1. Device Class


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BrianPugh/belay](https://github.com/BrianPugh/belay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
