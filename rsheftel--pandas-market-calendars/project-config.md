---
trigger: always_on
description: This document is a concise, practical guide for new contributors
---

# Pandas Market Calendars Developer Guidelines

This document is a concise, practical guide for new contributors

## Table of Contents

- [Context](#context)
- [Python Version](#python-version)
- [Planning Rules](#planning-rules)
- [Project Structure](#project-structure-high-level)
- [Domain: Market Calendars](#domain-market-calendars)
- [Common Patterns](#common-patterns)
- [Build and Configuration](#build-and-configuration)
- [Running Tests](#running-tests)
- [Contributing Tips](#contributing-tips)
- [Git Workflow](#git-workflow)
- [Changes](#changes)
- [Quick Checklist](#quick-checklist-for-new-contributions)
- [Code Style & Best Practices](#code-style--best-practices)
- [Import Style Rule](#import-style-rule)
- [Error Handling](#error-handling)
- [Naming](#naming)
- [Documentation](#documentation)
- [Pandas](#pandas)
- [Data Practices](#data-practices)
- [Testing Guidelines](#testing-guidelines)
- [Debugging Tips](#debugging-tips)
- [TODO Comments](#todo-comments)
- [Security](#security)
- [Performance](#performance)
- [When Uncertain](#when-uncertain)
- [Disallowed](#disallowed)

---

## Context

Pandas Market Calendars is a library for generating trading schedules for various exchanges.

You are an expert programming AI assistant who prioritizes minimalist, efficient code. You plan before coding, write
idiomatic solutions, seek clarification when needed, and accept user preferences even if suboptimal.

## Target Versions

- Python >= 3.8
- Pandas >= 2.3

## Planning Rules

- Create numbered plans before coding in the `.github/plans/` directory in the root of the project
- Display current plan step clearly
- Update the plan file as you progress
- Ask for clarification on ambiguity
- Optimize for minimal code and overhead
- After every major set of changes, generate a git commit message

## Project Structure (high level)

- Main library packages: `pandas_market_calendars/...`
- Tests in `tests/...` with test data in `tests/data`.
- Examples: `examples/...` for example code and tests.
- Documentation: `docs/...` for sources, `docs_build/...` for built docs.

## Domain: Market Calendars

- Core concepts: trading days, market hours, holidays, early closes
- Key classes: `MarketCalendar` (base class in `market_calendar.py`)
- Holiday definitions live in `pandas_market_calendars/holidays/`
- Calendar implementations in `pandas_market_calendars/calendars/`
- Calendar registration via `calendar_registry.py`
- Timezone handling: always use `pytz` or `zoneinfo`, never naive datetimes

## Common Patterns

### Adding a New Calendar

1. Create holiday definitions in `holidays/<exchange>.py`
2. Create calendar class in `calendars/<exchange>.py`
3. Register in `calendar_registry.py`
4. Add tests in `tests/test_<exchange>_calendar.py`

### Modifying Holiday Rules

- Historical holidays: add to existing rules with date ranges
- Future holidays: use observance rules where possible
- Test against known historical data in `tests/data/`

---

## Build and configuration

- Tooling
    - Package manager/runner: uv (Astral). Local workflows should prefer `uv` for env creation and running.
    - Python target: pyproject sets the target python with `requires-python =`, use the version in that file
    - Linting/format: Black only (see `[tool.black]` and `[tool.isort]` in `pyproject.toml`).

- Environment setup (local)
    - Minimal install (runtime deps):
        - `uv self update`
        - `uv sync --no-dev`
    - Full dev (adds linters, coverage, xdist, notebooks, profiling):
        - `uv sync --all-extras --group dev`
    - Windows: use PowerShell and backslashes in paths when invoking Python tools.

**MANDATORY RULE – DO NOT EVER BREAK THIS:**

This developer is using Git Bash (MSYS2) on Windows.

All shell commands, file paths, and code snippets **MUST**:

- Use only forward slashes `/` for paths
- Use only POSIX/bash commands
- Never use backslashes `\`, drive letters, or Windows-specific commands

Violating this rule makes the suggestion completely unusable.

## Running Tests

- Canonical commands
    - All tests: `uv run --no-sync pytest`
    - Fast suite (skip slow): `uv run --no-sync pytest -m "not slow"`
    - Quiet summary: `uv run --no-sync pytest -q`
    - Target a path/file: `uv run --no-sync pytest -q tests` or a single file

- Adding tests: conventions
    - All tests must pass without errors or warnings. If the tests produce warnings, modify the tests until they no
      longer produce warnings

## Contributing Tips

- Keep functions small, single-responsibility; group related code logically
- Write a matching pytest test file for any new module
- Add TODOs sparingly with concrete next steps
- Meaningful, atomic commits

## Git Workflow

- Branch naming: `feature/<description>`, `fix/<issue>`, `docs/<topic>`
- Commit messages: imperative mood, 50 char subject, blank line, body
- Always pull before starting work
- Squash commits for clean history when appropriate

## Changes

- The change log file is `docs/change_log.rst` file in this project, use that to track changes to the project.
- When *major* changes or features are made/added I want a concise summary of the change and files involved (summarize
  if too many files are changed). Use the change_log.rst file to track changes and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rsheftel/pandas_market_calendars](https://github.com/rsheftel/pandas_market_calendars) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
