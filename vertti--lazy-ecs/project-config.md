---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`lazy-ecs` is a CLI tool for working with AWS services, with initial focus on Amazon ECS. Built with modern Python tooling including uv for dependency management, ruff for linting/formatting, and pytest for testing.

## Development Environment

The project uses:

- `uv` for Python dependency management and virtual environments
- `mise` for tool management (Python, uv, Node.js)
- Python 3.11+ required
- `pyrefly` for fast Python type checking and inference

## Development Workflow (Test-Driven Development)

**This project follows a strict TDD approach with emphasis on small, testable functions:**

1. **Write a test** for the new feature/functionality
   - Prefer testing small, pure functions that take parameters
   - Avoid tests that require complex mocking when possible
2. **Run the test** to see it fail: `uv run pytest`
3. **Implement** the minimal code to make the test pass
   - Extract small, focused functions
   - Pass required data as parameters rather than using instance variables
4. **Run tests** again to verify they pass: `uv run pytest`
5. **Refactor** - extract more pure functions if the implementation is getting complex
6. **Format code**: `uv run ruff format` (fixes many linting issues automatically)
7. **Check and fix linting**: `uv run ruff check --fix` (fixes issues AND does the check)
8. **Type check**: `uv run pyrefly check` (fast type checking)
9. **Pause** - suggest commit message, never commit automatically

## Setup

After cloning the repository:

```bash
# Install dependencies
uv sync

# Install pre-commit hooks (automatically runs ruff on git commit)
uv run pre-commit install
```

## Common Commands

```bash
# Run the CLI
uv run lazy-ecs

# TDD Cycle Commands (in order):
uv run pytest                    # Run tests (should fail initially)
# [implement feature]
uv run pytest                    # Verify tests pass
uv run ruff format               # Auto-fix formatting (run first!)
uv run ruff check --fix          # Fix issues AND check linting with type annotation enforcement
uv run pyrefly check             # Fast type checking

# Type annotation tools:
uv run pyrefly infer             # Auto-add missing type annotations
uv run pyrefly check --verbose   # Detailed type checking output

# Other useful commands:
uv run pytest --cov             # Run tests with coverage
uv run pytest -v                # Verbose test output
uv run pytest tests/test_file.py # Run specific test file

# Pre-commit commands:
uv run pre-commit run --all-files # Run pre-commit on all files manually
```

**IMPORTANT**: This is a uv project - always prefix commands with `uv run` or `uv`.

## Project Structure

- `src/lazy_ecs/` - Main application code using Click for CLI
- `tests/` - Test files using pytest
- `pyproject.toml` - All configuration (dependencies, tools, build)

## Architecture

- **CLI Framework**: questionary + rich for interactive command-line interface
- **AWS Integration**: boto3 with boto3-stubs for type-safe AWS API interactions
- **Project Layout**: src-layout with lazy_ecs package
- **Testing**: pytest with coverage reporting and moto for AWS mocking
- **Code Quality**: ruff for linting and formatting
- **Type Checking**: pyrefly for fast Python type checking and inference
- **Entry Point**: `lazy-ecs` command installed as script

## Key Features

- Modern Python packaging with pyproject.toml
- Interactive CLI with arrow key navigation (questionary + rich)
- AWS ECS integration with boto3 and comprehensive typing
- Fast type checking with pyrefly and boto3-stubs
- Comprehensive test coverage with moto for AWS mocking
- All tooling configured in pyproject.toml

## Testing the Interactive UI

After implementing and testing a feature, verify it works correctly in the full interactive CLI:

```bash
# Test the interactive UI by piping keyboard inputs
printf '\n\033[B\n' | timeout 5 aws-vault exec working-aws-profile-name -- uv run lazy-ecs

# Key sequences:
# \n = Enter key (select)
# \033[B = Down arrow
# \033[A = Up arrow
# q = Quit (only works if Exit button is present)
```

**Important UI Verification Checklist:**

- ✅ Back button appears at bottom of selection menus
- ✅ Exit button appears at bottom of selection menus
- ✅ Navigation works correctly between screens
- ✅ Visual elements display correctly (no red text for healthy items)
- ✅ New features are accessible through the menu system

This allows testing the full navigation flow without manual interaction, ensuring features like navigation buttons haven't been accidentally removed during refactoring.

## Generating Feature Screenshots

Feature screenshots for the README are generated using Rich's built-in SVG export:

```bash
# Regenerate all screenshots
uv run python scripts/generate_screenshots.py
```

**How It Works:**

- `scripts/generate_screenshots.py` creates mock data and renders UI components
- Uses `Console(record=True)` to capture output
- Exports to SVG format in `images/` directory
- Screenshots are embedded in README.md

**When to Regenerate:**

- After visual UI improvements
- When adding new features that need screenshots

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vertti/lazy-ecs](https://github.com/vertti/lazy-ecs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
