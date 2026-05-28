---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a pytest plugin that enforces test timing constraints and validates test size distributions based on Google's "Software Engineering at Google" best practices. The plugin categorizes tests into four sizes (small, medium, large, xlarge) with specific time limits and tracks whether the test suite meets target distribution percentages.

## Agent Workflow Requirements

**CRITICAL: All agents working on this repository MUST follow these requirements:**

1. **GitHub Issue Required**: Create a GitHub issue for ALL work before starting implementation
   - Use descriptive titles and comprehensive descriptions
   - Add appropriate labels (bug, enhancement, documentation, etc.)
   - Reference related issues if applicable
   - Keep the issue updated with progress, blockers, and decisions

2. **Pull Request Required**: ALL changes MUST go through a Pull Request
   - Never commit directly to main branch
   - Create a feature branch for your work
   - Open a PR as soon as you have commits to share
   - Link the PR to the issue (use "Fixes #issue-number" in PR description)
   - Keep PR description updated with implementation details and testing notes

3. **Continuous Updates**: Keep issue and PR updated throughout development
   - Comment on the issue when starting work, encountering blockers, or making decisions
   - Update PR description as implementation evolves
   - Push commits frequently to keep PR current
   - Respond to review comments promptly

4. **Documentation Maintenance**: Keep documentation synchronized with code changes
   - Update relevant documentation in the SAME commit as code changes
   - This includes: README.md, CHANGELOG.md, docstrings, and this CLAUDE.md
   - Documentation is code - treat it with the same rigor
   - Never merge a PR with outdated documentation

5. **Book Content Updates**: This project is part of the "Effective Testing with Python" book ecosystem
   - Location: `/Users/mikelane/dev/effective-testing-with-python/`
   - When making **significant design decisions**, add an entry to `design-decisions/` using the template
   - When discovering **key insights or quotable content**, add to `notes/key-insights.md`
   - When changing **how tools integrate**, update `notes/tool-ecosystem.md`
   - Design decisions become book content — document the "why" not just the "what"
   - This is NOT required for routine bug fixes or minor changes

## Development Commands

### Installation
```bash
# Install all dependencies (development and production)
uv sync --all-groups
```

### Pre-commit Setup
```bash
# Install pre-commit hooks
uv run pre-commit install

# Run all pre-commit hooks manually
uv run pre-commit run --all-files
```

### Testing Commands

**Coverage Note**: This project uses `coverage run` instead of `pytest --cov` because pytest loads plugins before coverage tracking starts. Using `coverage run -m pytest` ensures all module-level code (imports, class definitions, decorators) is tracked correctly.

**Tox for Multi-Version Testing**: Use tox to test against all supported Python versions (3.11, 3.12, 3.13, 3.14) in isolated environments.

```bash
# Run tests across all Python versions with tox (full output, verbose)
uv run tox

# Run tests for a specific Python version
uv run tox -e py312

# Run tests in parallel across all versions (fast, minimal output)
uv run tox run-parallel

# Run fast parallel tests (used by pre-commit, with pytest-xdist)
uv run tox run-parallel -e py311-fast,py312-fast,py313-fast,py314-fast

# List all available tox environments
uv run tox list

# Run all tests with coverage (single version)
uv run coverage run -m pytest

# Run tests in parallel with pytest-xdist (single version)
uv run pytest -n auto

# View coverage report in terminal
uv run coverage report

# View detailed coverage with missing lines
uv run coverage report --show-missing

# Generate XML coverage report (for codecov)
uv run coverage xml

# Generate HTML coverage report for local viewing
uv run coverage html

# Run a single test file (without coverage)
uv run pytest tests/test_plugin_module.py

# Run a single test function (without coverage)
uv run pytest tests/test_plugin_module.py::test_function_name

# Run a specific test class (without coverage)
uv run pytest tests/test_plugin_module.py::DescribeClass

# Run tests with detailed output (without coverage)
uv run pytest -vv
```

### Code Quality
```bash
# Run isort (import sorting)
uv run isort .

# Run ruff check with auto-fix
uv run ruff check --fix .

# Run ruff format
uv run ruff format .
```

### Coverage Validation
```bash
# Check that coverage meets the target (100% by default)
uv run python tests/_utils/check_coverage.py
```

The coverage target is stored in `coverage_target.txt` (currently 100.0).

## Architecture

### Core Components

**Plugin Entry Point** (`src/pytest_test_categories/plugin.py`)
- Implements pytest hooks to integrate with pytest's test lifecycle
- Manages session state via `PluginState` (active status, timer, distribution stats, warnings, reports)
- Key hooks:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mikelane/pytest-test-categories](https://github.com/mikelane/pytest-test-categories) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
