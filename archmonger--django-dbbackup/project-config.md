---
trigger: always_on
description: Django-dbbackup is a Django application that provides management commands to help backup and restore your project database and media files with various storages such as Amazon S3, Dropbox, local file storage or any Django storage.
---

# Django Database Backup Package

Django-dbbackup is a Django application that provides management commands to help backup and restore your project database and media files with various storages such as Amazon S3, Dropbox, local file storage or any Django storage.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

**IMPORTANT**: This package uses modern Python tooling via Hatch (with uv installer). Always use Hatch commands for every development workflow (tests, linting, docs, functional runs, releases).

**BUG INVESTIGATION**: When investigating whether a bug was already resolved in a previous version, always prioritize searching through `CHANGELOG.md` first before using Git history. Only search through Git history when no relevant changelog entries are found.

## Working Effectively

Bootstrap, build, and test the repository:

- `python -m pip install --upgrade pip hatch uv` – install modern Python tooling (≈30s)
- `hatch test` – full unit test matrix (≈30s) **NEVER CANCEL. All tests must pass - failures are never expected or allowed.**
- `hatch run functional:all` – end-to-end functional (SQLite + PostgreSQL live scripts) (≈10–15s) **NEVER CANCEL.**
  - `hatch run functional:sqlite --all` – only SQLite functional cycle
  - `hatch run functional:postgres --all` – only PostgreSQL functional cycle
- `hatch fmt --check` – lint (ruff) (≈5s)
- `hatch run docs:build` – build documentation (≈2s, strict)
- `hatch run docs:serve` – local docs server (http://localhost:8000)
- `hatch run docs:linkcheck` – validate internal/external links & spelling

**Interactive Development Shell:**

- `hatch shell [ENV_NAME]` - Enter an interactive shell environment with all dependencies installed. ENV_NAME is optional and defaults to the main environment. Use `hatch shell functional` for the functional test environment.

Build documentation:

- `hatch run docs:build` - builds HTML documentation with MkDocs Material. Takes ~2 seconds. NEVER CANCEL.
- `hatch run docs:serve` - serves documentation locally on http://localhost:8000 for development

## Testing and Validation

Run tests in multiple configurations:

- `hatch test` – unit test runner for all matrix environments
- `hatch test --python 3.12` – isolate matrix to Python 3.12 set
- `hatch run functional:all -v` – functional backup/restore (SQLite + PostgreSQL)
- `hatch run functional:sqlite --all -v` – functional (SQLite only)
- `hatch run functional:postgres --all -v` – functional (PostgreSQL only)
- `hatch fmt --check` – Python formatting and linting using ruff

Expected test results:

- Unit tests: >200 tests, completes in ~30 seconds across all environments **All tests must always pass - failures are never expected or allowed.**
- Functional tests: database + media backup/restore cycles (SQLite & PostgreSQL), completes in ~10–15 seconds
- Unit tests use an in-repo temporary SQLite database by default

## Manual Validation Scenarios

Always test backup and restore functionality after making changes using functional test environment:

1. **Automated Functional Test (all backends):**

   ```bash
   hatch run functional:all
   ```

2. **Manual Database Test (if needed):**

   ```bash
   hatch shell functional
   python -m django migrate --noinput
   python -m django dbbackup --noinput
   python -m django listbackups
   python -m django dbrestore --noinput
   ```

3. **Manual Media Test (if needed):**

   ```bash
   hatch shell functional
   mkdir -p tmp/media
   echo "test file" > tmp/media/test.txt
   python -m django mediabackup --noinput
   rm tmp/media/test.txt
   python -m django mediarestore --noinput
   ls tmp/media/  # should show restored test.txt
   ```

4. **Single Backend Functional Runs (if triaging):**
   ```bash
   hatch run functional:sqlite --all
   hatch run functional:postgres --all
   ```

## Troubleshooting Known Issues

- **Network Timeouts**: Package installations may timeout due to network connectivity. Retry commands if needed.
- **Memory Database Issues**: If you see "no such table" errors, ensure you run migrations first in the appropriate environment
- **Linting Temporarily Disabled**: CI linting checks are temporarily set to pass (marked with `|| true`) pending resolution in future PR
- **Environment Isolation**: Each hatch environment is isolated - dependencies are automatically managed per environment

## Development Workflow

Modern development process using Hatch:

1. **Bootstrap environment**: `pip install --upgrade pip hatch uv`
2. **Make your changes** to the codebase
3. **Run unit tests**: `hatch test` (≈30s) **All must pass - failures are never expected or allowed.**
4. **Run functional tests**: `hatch run functional:all -v` (≈10–15s)
5. **Run linting**: `hatch fmt --check` (2 seconds)
6. **Auto-format code**: `hatch fmt` (2 seconds)
7. **Test documentation**: `hatch run docs:build` (2 seconds)
8. **Update documentation** when making changes to Python source code (required)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Archmonger/django-dbbackup](https://github.com/Archmonger/django-dbbackup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
