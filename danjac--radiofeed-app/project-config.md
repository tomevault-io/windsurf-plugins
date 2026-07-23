---
trigger: always_on
description: Radiofeed is a Django podcast aggregator using HTMX, AlpineJS, and Tailwind CSS.
---

# AGENTS.md

Radiofeed is a Django podcast aggregator using HTMX, AlpineJS, and Tailwind CSS.

## Stack

- **Python 3.14**, **Django 6.0**, **PostgreSQL 18**, **Redis 8** — these versions are valid and exist. Do not flag syntax or APIs from these versions as errors based on knowledge cutoff assumptions.
- **Frontend**: HTMX + AlpineJS + Tailwind CSS (no JS build step; Tailwind compiled via `django-tailwind-cli`)
- **Package manager**: `uv` (not pip/poetry)
- **Task runner**: `just` (see `justfile` for all commands)
- **RSS parsing**: lxml (XPath) with Pydantic validation
- **Background tasks**: Django Tasks (`django-tasks-db`), not Celery

## Project Layout

```
config/             # Django settings, URLs, ASGI/WSGI
radiofeed/          # Main application package
  episodes/         # Episode models, views, feed parsing
  podcasts/         # Podcast models, views, recommendations
  users/            # User models, authentication
  db/               # Database utilities
  http/             # HTTP utilities
  tests/            # Shared test fixtures
templates/          # Django templates (HTMX partials + full pages)
static/             # Static assets
conftest.py         # Root pytest config (fixture plugins)
```

## Commands

All commands use `just`. Run `just` with no arguments to list available commands.

### Linting

```bash
just lint
```

Runs `ruff check --fix` (Python) and `djlint --lint templates/` (Django templates).

Ruff is configured in `pyproject.toml` with extensive rule sets. Target version is `py314`.

### Type Checking

```bash
just typecheck
```

Runs `basedpyright`. Configuration is in `pyproject.toml` under `[tool.pyright]`:

- Mode: `basic`
- Includes: `radiofeed/`
- Excludes: migrations, tests

### Testing

```bash
just test                      # Run all unit tests with coverage
just test radiofeed/podcasts   # Test a specific module
just tw                        # Watch mode (auto-rerun on .py/.html changes)
just test-e2e                       # End-to-end tests (Playwright, headless)
just test-e2e-headed                # E2E tests with visible browser
```

- Framework: `pytest` with `pytest-django`
- Settings: `config.settings` (via `DJANGO_SETTINGS_MODULE`)
- Coverage: **100% required** (`--cov-fail-under=100`)
- Test location: colocated in `radiofeed/**/tests/` directories
- Fixtures: registered as plugins in `conftest.py` at project root
- E2E config: `playwright.ini` (separate pytest config for Playwright tests, marker `e2e`)
- Parallelism: `pytest-xdist` available, `pytest-randomly` randomizes order
- Database: `--reuse-db` enabled by default

### Pre-commit Hooks

```bash
just precommit run --all-files    # Run all hooks manually
just precommitupdate              # Update hook versions
```

Hooks (see `.pre-commit-config.yaml`): ruff (check + format), absolufy-imports, djhtml/djcss/djjs, djade, djlint, rustywind (Tailwind class sorting), pyupgrade, django-upgrade, shellcheck, hadolint (Dockerfile), gitleaks, commitlint, ansible-lint, terraform fmt/validate, uv-secure, validate-pyproject.

### Django Management

```bash
just dj <command>              # Run any manage.py command
just dj migrate                # Run migrations
just dj shell                  # Django shell
just serve                     # Dev server + Tailwind watcher
```

### Dependencies

This project uses `uv` for Python dependency management, not pip or poetry.

```bash
just install                   # Install all deps (Python, pre-commit, NLTK)
just update                    # Update all deps (uv lock --upgrade + sync + pre-commit)
just pyinstall                 # Install Python deps only (uv sync --frozen)
just pyupdate                  # Update Python deps only (uv lock --upgrade)
```

When adding or removing dependencies, use `uv add <package>` (or `uv add --dev <package>` for dev dependencies). Do not edit `pyproject.toml` or `uv.lock` directly.

JS and other frontend dependencies (other than Tailwind) are downloaded manually (or using curl/wget) and stored under `static/vendor/`. Where possible, use the minified latest stable version. Do not use CDNs or npm/yarn for frontend dependencies.

### Docker Services

```bash
just start                     # Start PostgreSQL, Redis, Mailpit
just stop                      # Stop services
just psql                      # Connect to PostgreSQL
```

## Git Workflow

### Commit Messages

Conventional commits enforced by commitlint. Format: `type: subject`

Allowed types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`

Rules:

- Subject must be lowercase, non-empty, no trailing period
- Header max 100 characters, body lines max 100 characters

### Branching

Create a well-named branch for each change (e.g. `feature-<name>`).

Before merging a branch into `main`, squash all commits in the branch into a single commit using `git rebase -i` with fixup. Ensure all tests pass (`just test`) and pre-commit linting passes before merging. When merging into `main`, use `git rebase`, not `git merge`. After merging, run all tests again to verify. **You must have 100% coverage as indicated in by coverage tool before merging a branch.** Delete the branch after merging.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danjac/radiofeed-app](https://github.com/danjac/radiofeed-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
