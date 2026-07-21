---
trigger: always_on
description: - Use `pip install -e .[dev]` to install the package in editable mode with
---

# Autonomous Coding Agents – Project Conventions
# This file documents conventions and workflows for agentic coding assistants
# (e.g., QQ, custom agents) that operate within this repository.
# It is intended to be used as a reference for building, linting, testing,
# and maintaining code quality. The file is deliberately verbose (~150 lines)
# to serve as a comprehensive guide for future agents.

## 1. Development Workflow
### Build / Install
- Use `pip install -e .[dev]` to install the package in editable mode with
  development dependencies (test, lint, type‑check tools).
- Keep a `requirements-dev.txt` that pins exact versions of linting and testing
  tools; regenerate with `pip freeze > requirements-dev.txt` when updating
  dependencies.
- For reproducible environments, a `Dockerfile` is provided; build with
  `docker build -t autonomous‑intelligence .`.

## Linting
- **ruff** is the primary linter/formatter.
  - Run `ruff check .` to perform static analysis and catch style violations.
  - Run `ruff format .` to auto‑format code; this command should be part of the
    pre‑commit hook.
- **black** is used for opinionated code formatting.
  - Run `black .` to reformat the entire codebase.
- **isort** orders imports; run `isort .` to enforce import ordering rules.
- Integrate both into the pre‑commit framework (see “Pre‑commit Hooks” below).

## Testing
- Primary test runner is **pytest**.
  - Execute `pytest` to discover and run all tests.
  - For CI pipelines, use `pytest --maxfail=1 --disable-warnings -q`.
- **Single‑test execution**:  
  - Syntax: `pytest <path/to/test_file.py>::<ClassOrFunction>::<method_name>`  
  - Example: `pytest tests/test_processing.py::ProcessTests::test_parsing`  
  - To run only the first failing test repeatedly:  
    ```bash
    while true; do
      pytest --maxfail=1 -x
      sleep 1
    done
    ```
- Use `--tb=short` to keep tracebacks concise; enable `--strict-metadata`
  to enforce type‑checking of test fixtures.

## Code Style Guidelines
### Imports
- Place standard‑library imports first, followed by third‑party, then internal
  imports.
- Use absolute imports; avoid wildcard (`*`) imports.
- Group imports by type and separate groups with a blank line.
- Prefer `import X` over `from X import *`.

### Formatting
- Max line length: **100 characters** (unless a clear reason exists).
- Trailing commas are required in multi‑line structures.
- End files with a single newline character.
- Use UTF‑8 encoding for all source files.

### File Structure
- Keep modules small and focused; each module should implement a single
  conceptual responsibility.
- Place tests in a parallel `tests/` directory; use descriptive file names.
- Keep configuration files (`pyproject.toml`, `.ruff.toml`, `.flake8`) at the
  repository root.

### Naming Conventions
- Functions, methods, and variables: `snake_case`.
- Classes: `PascalCase`.
- Constants: `ALL_CAPS`.
- Private helpers: prefix with `_`.
- Test classes/functions: prefix with `Test`/`test_`.

### Types & Annotations
- Add type hints to all public functions, methods, and class attributes.
- Use `typing.Union`, `typing.Optional`, and `typing.List` appropriately.
- For simple dicts, prefer `TypedDict`; for enums, use `Enum`.
- Avoid `Any`; if unavoidable, add a comment explaining why.

### Error Handling
- Define custom exception classes in a dedicated `exceptions.py` module.
- Raise specific exceptions rather than generic `Exception`.
- Do not use bare `except:`; always bind to a concrete exception type.
- Log errors with appropriate severity (`debug`, `info`, `warning`, `error`);
  avoid printing raw tracebacks to stdout.

### Logging
- Configure a module‑level logger: `logger = logging.getLogger(__name__)`.
- Use parameterized logging statements; include contextual data.
- In production, route logs to a structured log sink (e.g., JSON file);
  suppress `debug` level by default.

### Testing & Coverage
- Aim for **≥80 %** test coverage on newly modified modules.
- Use `pytest-cov` to generate coverage reports: `pytest --cov=.
- Enforce a minimum coverage threshold in CI (`--cov-fail-under=80`).
- Structure tests using fixtures and `@pytest.mark.parametrize` for reuse.

## 3. Dependency Management
- Use `pip-tools` to manage pinned dependencies in `requirements.in`.
- Run `pip-compile` to generate `requirements.txt`.
- Run `pip-sync` to ensure the environment matches `requirements.txt`.
- Keep `requirements-dev.in` for development‑only dependencies.

## 4. Pre‑commit Hooks
- Install with `pre-commit install`.
- Hooks include: `ruff`, `ruff-format`, `black`, `isort`, `mypy`, `bandit`,
  `pylint`.
- Run `pre-commit run --all-files` locally before committing.
- CI enforces that all hooks pass before merging.

## 5. Version Control & Collaboration
- Commit messages follow **Conventional Commits** (`feat:`, `fix:`, `docs:`…).
- Use feature branches; prefix branch names with `feature/`, `bugfix/`, or
  `refactor/`.
- Pull requests require at least **one** approving review; automated checks
  (lint, test, type) must pass.
- Keep CHANGELOG.md updated; use `towncrier` for version bumping.

## 6. CI / Continuous Integration
- GitHub Actions runs on every push and PR:
  - `lint` job runs ruff & black.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OriNachum/autonomous-intelligence](https://github.com/OriNachum/autonomous-intelligence) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
