---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Python package for converting Lanelet2 map format to OpenDRIVE format, designed for use with Autoware autonomous driving software. The project uses the modern Python packaging tool `uv` for dependency management and builds.

## Development Environment Setup

This project uses `uv` (version 0.9.7+) for Python package management. Ensure uv is installed before working with this codebase.

### Key Commands

```bash
# Install dependencies
uv pip install -e .

# Sync dependencies from lock file
uv sync

# Add a new dependency
uv add <package_name>

# Add a development dependency
uv add --dev <package_name>

# Run Python scripts
uv run python <script.py>

# Create virtual environment (if needed)
uv venv
```

## Local Test Verification (Container-Based)

**IMPORTANT**: Run local test verification inside the Docker container, not directly on the host.

### Why

The runtime dependency `lanelet2-python-api-for-autoware` is built from source against system Boost. Many host environments (e.g., Ubuntu 24.04 with Boost 1.83) cannot compile it — `uv sync` and `uv run pytest` fail with `RuntimeError: Command failed: make -j24` during the wheel build. The Docker image pins Ubuntu 22.04 with Boost 1.74, matching CI exactly, and avoids this failure.

### How

The repository ships a multi-stage `Dockerfile` and `docker-compose.yml` with profiles that mirror each CI job. See [`docs/docker.md`](docs/docker.md) for the full reference. The most common commands:

```bash
# Run the full pytest suite (matches CI's `test` job)
docker compose --profile test run --rm pytest

# Run pre-commit on all files (matches CI's `lint-and-format` job)
docker compose --profile lint run --rm lint

# Open an interactive shell with the workspace bind-mounted
docker compose --profile dev run --rm dev
```

### Instructions for Claude Code

When the user asks to "run the tests", "verify locally", or otherwise validate a change end-to-end:

1. **Do NOT run `uv run pytest` on the host.** It will likely fail on the Boost build step, producing noise unrelated to the change.
2. **Use `docker compose --profile test run --rm pytest`** for the full suite, or the appropriate profile (`lint`, `qc`, `carla`) for a narrower check.
3. If the container is unavailable in the current environment (e.g., Docker not installed), say so explicitly rather than running broken host commands. Defer test verification to CI in that case.
4. Static checks that do **not** import the package (e.g., `ruff`, `ruff-format`, `mypy --ignore-missing-imports` on individual files) **can** still be run on the host and should be used for fast iteration.

### Rationale

- **Reproducibility**: Container matches CI exactly; host doesn't.
- **Avoids false negatives**: Host build failures are an environment quirk, not a code defect.
- **Fast iteration**: Static checks on the host stay fast; dynamic test verification gets pushed to a known-good environment.

## Pre-commit Hooks and Lint Checking

**CRITICAL**: This project uses pre-commit hooks to ensure code quality and prevent lint errors. All commits must pass these checks before being pushed.

### Installation

Pre-commit hooks must be installed in your local repository before making any commits:

```bash
# Install pre-commit (if not already installed)
pip install pre-commit

# Install the git hook scripts
pre-commit install
```

### Usage

#### Automatic Checking (Recommended)
Once installed, pre-commit hooks run automatically on every `git commit`. The commit will be blocked if any checks fail.

#### Manual Checking
You can manually run pre-commit checks before committing:

```bash
# Run on all files
pre-commit run --all-files

# Run on staged files only
pre-commit run

# Run on specific files
pre-commit run --files <file1> <file2>
```

### Common Lint Errors and Fixes

If pre-commit hooks fail:

1. **Review the error messages** - They usually indicate what needs to be fixed
2. **Let pre-commit auto-fix when possible** - Many formatters (like black, isort) automatically fix issues
3. **Stage the auto-fixed changes**:
   ```bash
   git add -u
   ```
4. **Retry the commit**:
   ```bash
   git commit
   ```

### Important Notes for Claude Code

**MANDATORY**: When working with this repository through Claude Code:

1. **Always install pre-commit hooks** at the start of any work session:
   ```bash
   pre-commit install
   ```

2. **Never bypass pre-commit hooks** with `--no-verify` flag (this is already prohibited in Git Operation Restrictions)

3. **CRITICAL: Auto-format code BEFORE committing** to prevent CI/CD failures:
   ```bash
   # Step 1: Run pre-commit on all files to auto-fix formatting issues
   pre-commit run --all-files

   # Step 2: If files were modified, stage the changes
   git add -u

   # Step 3: Now commit (pre-commit will pass because code is already formatted)
   git commit -m "your message"
   ```

   **Rationale**: GitHub Actions fails when pre-commit hooks modify files (exit code 1). By running `pre-commit run --all-files` before committing, formatters like `ruff-format` will fix issues locally first, preventing CI failures.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tier4/autoware_lanelet2_to_opendrive](https://github.com/tier4/autoware_lanelet2_to_opendrive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
