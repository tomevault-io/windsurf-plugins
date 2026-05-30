---
trigger: always_on
description: This is a Python development project that prioritizes code quality, maintainability, and reliable testing. Your role is to help develop robust, well-tested code that follows established patterns and best practices. The project uses modern Python tooling with uv as the package manager.
---

# Python Development Guidelines

<project_context>
This is a Python development project that prioritizes code quality, maintainability, and reliable testing. Your role is to help develop robust, well-tested code that follows established patterns and best practices. The project uses modern Python tooling with uv as the package manager.
</project_context>

## Core Development Principles

<python_execution>
**CRITICAL**: When executing Python code, the assistant must NEVER invoke Python directly. Instead, the assistant must ALWAYS use `uv run` to ensure proper environment isolation and dependency management.

<incorrect_approach>
**FORBIDDEN - Never use this:**
```bash
python - <<'PY'
import sys
print(f"Running on Python {sys.version}")
PY
```
</incorrect_approach>

<correct_approach>
**REQUIRED - Always use this:**
```bash
uv run python - <<'PY'
import sys
print(f"Running on Python {sys.version}")
PY
```
</correct_approach>
</python_execution>

## Development Guidelines

### Package Management with uv

<package_management>
<motivation>
We use uv exclusively for package management because it provides consistent, reproducible environments and faster dependency resolution than traditional tools.
</motivation>

<instructions>
1. **Install packages**: `uv add package`
2. **Run tools**: `uv run tool`
3. **Upgrade packages**: `uv add --dev package --upgrade-package package`
4. **Sync dependencies**: `uv sync` (run this frequently to ensure consistency)
5. **Always use uv commands** for any package operations
</instructions>
</package_management>

### Code Quality Standards

<code_quality>
<motivation>
High-quality code reduces bugs, improves maintainability, and makes collaboration easier. These standards help ensure our codebase remains professional and scalable.
</motivation>

<requirements>
- **Type hints**: Include for all function parameters and return values
- **Docstrings**: Write for all public APIs following Google style
- **Function design**: Keep functions focused and single-purpose
- **Consistency**: Follow established patterns in the existing codebase
- **Line limits**: Format code with 88-character line limits for readability
</requirements>

<formatting_guidelines>
When lines exceed 88 characters:
- **Strings**: Wrap using parentheses for implicit concatenation
- **Function calls**: Use multi-line format with proper indentation
- **Imports**: Split into multiple lines with one import per line
</formatting_guidelines>
</code_quality>

### Testing Excellence

<testing>
<motivation>
Comprehensive testing catches bugs early, documents expected behavior, and enables confident refactoring. Well-written tests serve as living documentation of how code should behave.
</motivation>

<testing_approach>
- **Framework**: Execute tests with `uv run pytest`
- **Coverage**: Write tests that cover edge cases and error conditions
- **New features**: Include tests for all new functionality
- **Bug fixes**: Add regression tests when fixing bugs
- **Migration note**: Tests in `tests_old/` need rewriting to modern standards
</testing_approach>
</testing>

### Version Control Best Practices

<version_control>
<commit_messages>
Write clear, descriptive commit messages that explain the change and its purpose.

**For user-reported issues**, acknowledge the reporter:
```bash
git commit --trailer "Reported-by: <user_name>"
```

**For GitHub issues**, reference the issue number:
```bash
git commit --trailer "Github-Issue: #<number>"
```

Focus commit messages on describing the problem solved and the approach taken.
</commit_messages>

<pull_requests>
- Write detailed PR descriptions focusing on:
  - The problem being solved
  - The high-level approach taken
  - Any important design decisions
- Add **Tom and Bob** as reviewers for all PRs
- Keep PR descriptions focused on the change itself, not the tools used
</pull_requests>
</version_control>

## Development Workflow

### Environment Setup

<setup>
Initialize your environment with:
```bash
uv lock && uv sync --group dev && uv run pre-commit install
```

This will:
1. Refresh the lockfile
2. Install the dev group into `.venv`
3. Register the hooks in the managed environment

**Note**: `uv run` automatically locks and syncs before each command. Pass `--locked`/`--frozen` in CI if you need deterministic lockfiles.
</setup>

### Code Formatting

<formatting>
Use **Ruff** for consistent Python formatting (scoped to `src/` and `tests/`):

<commands>
- **Format code**: `uv run ruff format src tests`
- **Check style**: `uv run ruff check src tests`
- **Apply fixes**: `uv run ruff check src tests --fix`
</commands>

<key_considerations>
- **Line length**: 88 characters maximum
- **Import sorting**: Follows isort conventions (I001)
- **Clean imports**: Remove unused imports for clean code
</key_considerations>
</formatting>

### Justfile Shortcuts

<justfile>
Prefer the project Justfile for common routines:
- `just format` / `just lint` / `just style` - Ruff formatting and checks
- `just typecheck` - Run `uv run ty check .`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [abazabaaa/agents_experiments](https://github.com/abazabaaa/agents_experiments) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-30 -->
