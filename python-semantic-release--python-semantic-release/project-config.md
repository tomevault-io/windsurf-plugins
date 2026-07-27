---
trigger: always_on
description: This document explains how GitHub Copilot-like automated agents should interact with
---

# Copilot Instructions for python-semantic-release

This document explains how GitHub Copilot-like automated agents should interact with
the python-semantic-release repository.

## Project Overview

Python Semantic Release is a tool for automating semantic versioning and marking releases for
various types of software projects. It analyzes commit messages with various commit parsers
(the most notable being the Conventional Commits specification) to determine what the next
version should be and facilitates release steps that the developer generally has to do. This
includes generating changelogs, stamping the code with version strings, creating a repository
tag and annotating releases on a remote Git server with version-specific release notes.

**Key Components:**
- **CLI**: Command-line interface for version management, changelog generation, and publishing
- **Commit Parsers**: Parse commit messages to determine version bumps
  (Supports Conventional-Commits, Emoji, and Scipy format)
- **HVCS Integration**: Integrations with GitHub, GitLab, Gitea, and Bitbucket for releasing
- **Version Management**: Semantic versioning logic and version calculation
- **Changelog Generation**: Automated and customizable changelog creation using Jinja2 templates

## Development Setup

### Installation

Requires 3.9+ for development dependencies, but runtime supports 3.8+.

```bash
# Set up for development
pip install -e .[build,dev,docs,mypy,test]
```

### Running the Application

```bash
# See the CLI help
semantic-release --help

# Common commands
semantic-release version
semantic-release changelog
semantic-release publish
```

### Making Changes

Minimal PR checklist (run locally before proposing a PR):

- [ ] Run pre-PR checklist script (see below)
- [ ] If you added dependencies: update `pyproject.toml` and mention them in the PR.
- [ ] Review the helpful tips at the bottom of this document to ensure best practices.
- [ ] Verify that commit messages follow the Commit Message Conventions section below.

Runnable pre-PR checklist script (copyable):

```bash
# lint & format
ruff format .
ruff check --unsafe-fixes .
# run type checks
mypy .
# run unit tests
pytest -m unit
# run e2e tests
pytest -m e2e
# optional docs build when docs changed
sphinx-build -b html docs docs/_build/html
```

## Code Style and Quality

### Linting and Formatting

- **Ruff**: Primary linter and formatter (replaces Black, isort, flake8)

  ```bash
  # run check for lint errors
  ruff check --unsafe-fixes .

  # apply lint fixes
  ruff check --unsafe-fixes --fix .

  # check for formatting issues
  ruff format --check .

  # apply formatting fixes
  ruff format .
  ```

- **Type Checking**: Use mypy for type checking

  ```bash
  mypy .
  ```

### Code Style Guidelines

1. **Type Hints**: All functions must have complete type hints (enforced by mypy)

2. **Docstrings**: Use sphinx-style docstrings (though currently many are missing - add
   only when modifying a function or adding new code)

3. **Line Length**: 88 characters (enforced by Ruff)

4. **Import Style**:

   - Absolute imports only (no relative imports)
   - All files must use `from __future__ import annotations` for ignoring type hints at runtime
   - Prefer `from module import Class` over `import module` when using classes/functions
   - Running Ruff with `--unsafe-fixes` and `--fix` will automatically sort and group imports
   - All files should have a `if TYPE_CHECKING:  # pragma: no cover` block for type-only imports
   - Prefer renaming `re` imports for clarity (e.g. `from re import compile as regexp, escape as regexp_escape`)

5. **String Quotes**: Use double quotes for strings

6. **Error Handling**: Create specific exception classes inheriting from `SemanticReleaseBaseError`
   and defined in `errors.py`

### Common Patterns

- Configuration uses Pydantic models (v2) for validation
- CLI uses Click framework with click-option-group for organization
- Git operations use GitPython library
- Templating uses Jinja2 for changelogs and release notes

## Testing

### Test Structure

- **Unit Tests**: `tests/unit/` - Fast, isolated tests

- **E2E Tests**: `tests/e2e/` - End-to-end integration tests performed on real git repos
  (as little mocking as possible, external network calls to HVCS should be mocked). Repos are
  cached into `.pytest_cache/` for faster test setup/runs after the first build. E2E tests are
  built to exercise the CLI commands and options against real git repositories with various commit
  histories and configurations.

- **Fixtures**: `tests/fixtures/` - Reusable test data and fixtures

- **Repository Fixtures**: `tests/fixtures/repos/` - Example git repositories for testing and rely on
  `tests/fixtures/example_project.py` and `tests/fixtures/git_repo.py` for setup

- **Monorepo Fixtures**: `tests/fixtures/monorepo/` - Example monorepos for testing monorepo support

- **GitHub Action Tests**: `tests/gh_action/` - Tests for simulating GitHub Docker Action functionality

### Running Tests

```bash
# Run only unit tests
pytest -m unit

# Run only e2e tests
pytest -m e2e

# Run comprehensive (unit & e2e) test suite with full verbosity (all variations of repositories)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [python-semantic-release/python-semantic-release](https://github.com/python-semantic-release/python-semantic-release) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
