---
trigger: always_on
description: This document provides guidelines for AI agents (e.g., Claude Code agents) working on the mcpbr project. Following these guidelines ensures consistent, high-quality contributions.
---

# Agent Guidelines for mcpbr Development

This document provides guidelines for AI agents (e.g., Claude Code agents) working on the mcpbr project. Following these guidelines ensures consistent, high-quality contributions.

## Table of Contents

1. [Virtual Environment Setup](#virtual-environment-setup) **← START HERE**
2. [Pre-Commit Checklist](#pre-commit-checklist)
3. [Code Quality Requirements](#code-quality-requirements)
4. [Testing Requirements](#testing-requirements)
5. [Documentation Requirements](#documentation-requirements)
6. [Pull Request Guidelines](#pull-request-guidelines)

## Virtual Environment Setup

**⚠️ CRITICAL: ALWAYS USE A VIRTUAL ENVIRONMENT**

macOS uses system Python protection (PEP 668) which prevents installing packages globally. **DO NOT** use `--break-system-packages` or try to install packages system-wide. Instead, ALWAYS use a virtual environment or `uv`/`uvx`.

### Option 1: Using uv (Recommended)

The project uses `uv` for dependency management. Use `uvx` for running commands and `uv run` for scripts:

```bash
# Run commands without installation
uvx ruff check src/ tests/
uvx ruff format src/ tests/

# Run Python scripts/tests
uv run pytest -m "not integration"

# Install project in development mode (if needed)
uv pip install -e ".[dev]"
```

**This is the preferred approach** as it's already configured in the project.

### Option 2: Traditional Virtual Environment

If you need a traditional venv:

```bash
# Create virtual environment (one time)
python3 -m venv .venv

# Activate it (REQUIRED for every terminal session)
source .venv/bin/activate

# Install dependencies
pip install -e ".[dev]"

# Now you can use pip and run commands normally
pip install pre-commit
pytest -m "not integration"

# Deactivate when done
deactivate
```

### Option 3: Using pipx for Global Tools

For global tools like `pre-commit`, use `pipx`:

```bash
# Install pipx (one time)
brew install pipx

# Install global tools
pipx install pre-commit
pipx install pytest
```

### Checking Active Environment

Before running Python commands, verify your environment:

```bash
# Check if in a virtual environment
echo $VIRTUAL_ENV

# Should show path like: /Users/you/Projects/mcpbr/.venv

# Check Python location
which python3
# Should NOT be: /usr/bin/python3 (system Python)
# Should be: .venv/bin/python3 (venv) or managed by uv
```

### Common Errors and Solutions

**Error: `externally-managed-environment`**
```bash
# ❌ DON'T: Try to install globally
pip install something

# ✅ DO: Use uv or venv
uvx something  # for CLI tools
uv run python script.py  # for scripts
# OR activate venv first
source .venv/bin/activate && pip install something
```

**Error: `command not found: pytest`**
```bash
# ❌ DON'T: Install globally
pip install pytest

# ✅ DO: Use uv run or activate venv
uv run pytest  # preferred
# OR
source .venv/bin/activate && pytest
```

### Project Setup Workflow

When starting work on this project:

```bash
# 1. Clone the repository
git clone https://github.com/greynewell/mcpbr.git
cd mcpbr

# 2. Choose your environment approach:

# Option A: Using uv (recommended)
# No setup needed! Just use `uvx` and `uv run` commands

# Option B: Using venv
python3 -m venv .venv
source .venv/bin/activate
pip install -e ".[dev]"

# 3. Install pre-commit hooks (one time)
pipx install pre-commit  # if not already installed
pre-commit install

# 4. Verify setup
uv run pytest --version  # Should work
uvx ruff --version  # Should work
```

### Remember

- **NEVER** install Python packages system-wide on macOS
- **ALWAYS** use `uv run` / `uvx` OR activate venv first
- **CHECK** your environment with `echo $VIRTUAL_ENV` when debugging
- The project is configured to use `uv` - prefer `uvx` and `uv run` commands

## Pre-Commit Checklist

**MANDATORY:** Before committing code or creating a pull request, agents MUST complete ALL of the following steps:

### 1. Run Linting

```bash
# Check for linting issues
uvx ruff check src/ tests/

# Auto-fix fixable issues
uvx ruff check --fix src/ tests/

# Format code
uvx ruff format src/ tests/

# Verify all issues are resolved
uvx ruff check src/ tests/
```

**Expected output:** `All checks passed!`

If any linting errors remain, they MUST be fixed manually before proceeding.

**Quick one-liner:**
```bash
uvx ruff check --fix src/ tests/ && uvx ruff format src/ tests/ && uvx ruff check src/ tests/
```

### 2. Run Type Checking

```bash
# Run mypy on source code
uv run mypy src/mcpbr/
```

**Expected output:** `Success: no issues found`

If any type errors remain, they MUST be fixed before proceeding.

### 3. Run Tests

```bash
# Run all non-integration tests
uv run pytest -m "not integration"

# For integration tests (if applicable)
uv run pytest -m integration
```

**Expected result:** All tests must pass with 0 failures.

### 4. Update CHANGELOG

**MANDATORY:** If your changes are user-visible, update CHANGELOG.md:

```bash
# Edit CHANGELOG.md and add entry under [Unreleased]
# Example:
# ### Fixed
# - Fixed XYZ issue (#123)

# Verify entry is properly formatted
cat CHANGELOG.md | head -30
```

### 5. Verify Changes

- Review all modified files

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [greynewell/mcpbr](https://github.com/greynewell/mcpbr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
