---
trigger: always_on
description: Odin is a Django-based IoT dashboard for sensor management, weather monitoring, and home automation.
---

# AGENTS.md

## Project Overview

Odin is a Django-based IoT dashboard for sensor management, weather monitoring, and home automation.
It uses Django 5.2.7 with Django REST Framework, PostgreSQL, Redis, and modern Python tooling.

## Project Structure

- All REST API related code placed in `./odin/api/`
- Django apps with models, admin classes, migrations and management commands in `./odin/apps/`
- Tests in `./odin/tests/`
- Static files like images, CSS and JS in `./odin/static/`
- Django templates in `./odin/templates/`

## Git Workflow

This project adheres strictly to the Git Flow branching model. AI agents must follow these guidelines:

### Main Branch:

- The `master` branch always contains production-ready, stable code.
- Never commit directly to `master`.
- Do not use `git push --force` on the `master` branch.
- Do not merge branches into `master` without explicit approval.

### Feature Branches:

- Create feature branches using the naming convention `<agent-name>/feature/<issue-id>-<descriptive-name>`, where `<agent-name>` can be `opencode`, `cursor`, `copilot`, etc.
- Use the [Conventional Commits](https://www.conventionalcommits.org) specification for commit messages (e.g., `feat:`, `fix:`, `docs:`).
- Ensure all local tests pass before committing.
- Use `git push --force-with-lease` if needed on your feature branch, but never on `master`.

### Pull Requests (PRs):

- Open a Pull Request for every completed feature branch.
- PRs must be reviewed and pass all CI checks before merging.
- The PR title should follow the Conventional Commits specification.

## Linear Workflow

- When starting implementation of any issue from `TODO`, move it to `In Progress` column.
- When feature is completed and PR is created, move it to `In Review` column.
- After approval, merge the feature branch into `master` and move the issue to `Done` column.
- If the feature branch is not merged into `master`, move it back to `In Progress` column.
- If the feature branch is closed without merging, move it to `Closed` column.

## Development Commands

### Package Management

```bash
uv sync                    # Install dependencies
uv sync --upgrade          # Upgrade dependencies
```

### Django Management

```bash
uv run manage.py runserver                  # Start development server
uv run manage.py migrate                    # Run database migrations
uv run manage.py makemigrations             # Create new migrations
uv run manage.py collectstatic --no-input   # Collect static files
uv run manage.py makemessages -l ru         # Create translation files
uv run manage.py compilemessages -l ru      # Compile translation files
```

### Database Operations

```bash
# Development
make dump     # Backup database to odin.sql
make restore  # Restore database from odin.sql
make migrate  # Run database migrations
make migrations  # Create new migrations
make locale   # Compile translation files

# Django checks
uv run manage.py makemigrations --dry-run --check --verbosity=3 --settings=odin.settings.sqlite
uv run manage.py check --fail-level WARNING --settings=odin.settings.sqlite
```

### Testing

```bash
# Run all tests
uv run pytest --create-db --disable-warnings --ds=odin.settings.test odin/

# Run single test file
uv run pytest --create-db --disable-warnings --ds=odin.settings.test odin/tests/api/test_sensors.py

# Run single test method
uv run pytest --create-db --disable-warnings --ds=odin.settings.test odin/tests/api/test_sensors.py::TestSensorsAPI::test_sensors__list

# Run with coverage (if available)
uv run pytest --cov=odin --cov-report=term-missing odin/
```

### Code Quality

```bash
# Run all pre-commit hooks
uv run pre-commit run

# Individual tools
uv run ruff check .                 # Lint
uv run ruff format .                # Format
uv run bandit -c pyproject.toml .   # Security analysis
```

## Language & Environment

- Follow PEP 8 style guidelines, use Ruff for formatting (120 char line length)
- Use type hints for all function parameters and return values
- Prefer f-strings for string formatting over .format() or %
- Use list/dict/set comprehensions over map/filter when readable
- Prefer Pathlib over os.path for file operations
- Follow PEP 257 for docstrings (simple summary line plus detailed explanation)
- Use structural pattern matching (match/case) for complex conditionals
- Prefer EAFP (try/except) over LBYL (if checks) for Python idioms
- Prefer round brackets instead of square brackets where it possible (tuples instead of lists)

## Django Framework

- Follow Django best practices and the MVT pattern
- Use Django ORM effectively, avoid raw SQL
- Use select_related and prefetch_related to avoid N+1 queries
- Never modify migrations after deployment
- Class-based views for API, function-based views for Django apps views
- Put business logic to services, prefer functions, models and views should be short and clean

### Background Tasks

- Use RQ for long-running or resource-intensive operations
- Define jobs as additional functions with prefix `queue_`
- Configure queue names for different priority levels

```python
import django_rq


def sync_sensor_data(sensor_id: str):
    # Long-running sync operation
    return sensor_id



<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [manti-by/odin](https://github.com/manti-by/odin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
