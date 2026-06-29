---
trigger: always_on
description: This document is a concise, practical guide for new contributors
---

# Raccoon Developer Guidelines

This document is a concise, practical guide for new contributors

## Context

Raccoon is a Python library for data analysis that aims to make data structures similar to pandas Series and DataFrames,
but an emphasis performance especially for inreasing the size of existing datasets. In particular, the speed of
appending new rows to a DataFrame is critical for many real-world use cases.

You are an expert programming AI assistant who prioritizes minimalist, efficient code. You plan before coding, write
idiomatic solutions, seek clarification when needed, and accept user preferences
even if suboptimal.

## Target Versions

- Python >= 3.13

## Planning Rules

- Create numbered plans before coding in the `.github/plans/` directory in the root of the project
- Display current plan step clearly
- Update the plan file as you progress
- Ask for clarification on ambiguity
- Optimize for minimal code and overhead
- After every major set of changes, generate a git commit message

## Project Structure (high level)

- Main library packages: `raccoon/...`
- Tests in `tests/...`
- Examples: `examples/...` for example code and tests.
- Documentation: `docs/...` for sources, `docs_build/...` for built docs.

## Build and configuration

**MANDATORY RULE – DO NOT EVER BREAK THIS:**

This developer is using Git Bash (MSYS2) on Windows.

All shell commands, file paths, and code snippets **MUST**:

- Use only forward slashes `/` for paths
- Use only POSIX/bash commands
- Never use backslashes `\`, drive letters, or Windows-specific commands

Violating this rule makes the suggestion completely unusable.

- Tooling
    - Package manager/runner: uv (Astral). Local workflows should prefer `uv` for env creation and running.
    - Python target: pyproject sets the target python with `requires-python`, use the version in that file
    - Linting/format: Ruff only (see `[tool.ruff]` and `[tool.ruff.lint]` in `pyproject.toml`).

- Environment setup (local)
    - Minimal install (runtime deps):
        - `uv self update`
        - `uv sync --no-dev`
    - Full dev (adds linters, coverage, xdist, notebooks, profiling):
        - `uv sync --all-extras --group dev`
    - Windows: use PowerShell and backslashes in paths when invoking Python tools.

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
  as a template.

## Quick checklist for new contributions

- [ ] Create/activate env via uv; run `uv sync --group dev` if you need lint/coverage/xdist.
- [ ] Always do a git pull when starting a new chat on a project that has a git repo.
- [ ] Add focused tests next to code, with assets under `tests/data` pattern
- [ ] Run `uv run --no-sync pytest -m "not slow"` locally; consider `-n auto`.
- [ ] Run `uv run python -m ruff .` on any python files you have edited.
- [ ] Confirm touched Python files do not introduce new editor or type-checker diagnostics.
- [ ] Document assumptions (columns/dtypes, units, timezones).

---

## Code Style & Best Practices

### General Principles

- Follow PEP 8 with max line length 120 (Ruff enforces formatting and import sorting when enabled)
- Always prioritize readability and clarity
- Write concise, efficient, and idiomatic code
- Avoid duplicate code
- Prefer composition over inheritance

### Functions & Structure

- Keep functions short, focused on a single responsibility
- Prefer functions in modules rather than OOP, though classes are fine when they make sense
- Avoid deep nesting: limit to 2-3 levels of indentation; refactor complex logic into smaller functions
- Group related code logically

### Immutability & State

- Favor immutability: return new variables instead of modifying inputs (e.g., `df = df.copy()`)
- Never mutate global state; functions should be pure if possible
- Avoid: mutable default args, singleton globals, shared mutable state

### Type Hints & Syntax

- Always include type hints
- Use built-in generics (`list[str]`, `dict[str, int]`, `set[str]`) and `| None` for optionals; avoid `List`, `Dict`,
  `Optional` from typing module

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rsheftel/raccoon](https://github.com/rsheftel/raccoon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
