---
trigger: always_on
description: **Before proceeding with any work, please read these files in order:**
---

# AGENTS.md

## Getting Started - Required Reading

**Before proceeding with any work, please read these files in order:**

1. **README.md** - Project overview and setup instructions
2. **docs/development.md** - Development environment setup guide
3. **docs/contributing.md** - Contribution guidelines

## Project Overview

EDA Server (Event-Driven Ansible Controller) is a REST API application that provides event-driven automation capabilities. It manages rulebooks, activations, credentials, decision environments, and event streams.

**Key Technologies** (refer to `pyproject.toml` for specific package versions):
- Django + Django REST Framework
- Django Channels + Daphne (WebSocket support)
- PostgreSQL
- dispatcherd (PostgreSQL-based task queue)
- django-ansible-base (DAB) for RBAC, JWT auth, resource registry
- Poetry for dependency management
- Taskfile (task/go-task) for build automation
- pytest for testing

**Related Repositories:**
- `django-ansible-base` - Shared RBAC, JWT, and feature flag library used across the platform
- `ansible-rulebook` - Rulebook engine for event-driven automation
- `ansible-runner` - Ansible execution engine
- `dispatcherd` - PostgreSQL-based task dispatcher
- `eda-server-operator` - Kubernetes operator for production deployment

## Build and Test Commands

### Initial Setup

Check `pyproject.toml` for supported Python versions. Always use the most recent supported version.

```bash
# Install dependencies and pre-commit hooks
task dev:init

# Start PostgreSQL and Redis (required for tests)
task docker:up:minimal

# Run database migrations
task manage -- migrate

# Seed initial data
task manage -- create_initial_data

# Create admin superuser (admin/testpass)
task create:superuser

# Start API server
task run:api
```

### Running Tests

**IMPORTANT:** Tests require PostgreSQL to be running. Start it with `task docker:up:minimal` first.

```bash
# Run all tests (runs twice: normal + multithreaded)
task test

# Run a specific test module
task test -- tests/integration/api/test_activation.py

# Run a single test
task test -- tests/integration/api/test_activation.py::test_retrieve_activation

# Run tests with Docker
task docker -- run --rm eda-api python -m pytest

# Run with verbose output
task test -- tests/integration/api/test_activation.py -v
```

**Test Markers:**
- `@pytest.mark.multithreaded` - Tests that must run in a separate process (run automatically by `task test`)

**Test Configuration:**
- pytest config is in `pytest.ini`
- `asyncio_mode = auto` (pytest-asyncio)
- `DJANGO_SETTINGS_MODULE = aap_eda.settings.default`
- Tests live in `tests/integration/` and `tests/unit/`

### Linting and Formatting

```bash
# Run all linters
task lint

# Individual linters
task lint:black       # Code formatting check
task lint:isort       # Import sorting check
task lint:ruff        # Fast linter (E, F, D, TID rules)
task lint:flake8      # Traditional linting
task lint:migrations  # Django migration check

# Auto-format code
task format           # Runs isort + black
```

## Code Style Guidelines

**Pre-commit Hooks** (installed via `task dev:init`):
- **black**: Code formatting (line length: 79)
- **isort**: Import sorting (black profile, line length: 79)
- **ruff**: Fast linter
- **flake8**: Traditional linting with plugins

**Testing Conventions:**
- Integration tests go in `tests/integration/` mirroring the source structure (`api/`, `core/`, `services/`, `tasks/`, `wsapi/`)
- Unit tests go in `tests/unit/`
- Global fixtures are defined in `tests/conftest.py`
- API tests use DRF's `APIClient`
- Use descriptive test names that explain what is being tested
- Include both positive and negative test cases
- Use `@pytest.mark.parametrize` for similar test cases

## Security Considerations

- **SonarCloud**: Code quality and security analysis integrated via CI
- **Codecov**: Coverage tracking
- **Credential Storage**: All credentials are encrypted (via Cryptography + GPG)
- **Authentication**: JWT-based via django-ansible-base
- **Pre-commit Hooks**: Always run to catch security and quality issues

### GitHub Actions Security
- **NEVER use user-controlled data directly in run blocks**
  - Always pass through environment variables (e.g., `github.event.pull_request.body`)
- **Bad:** `echo "${{ github.event.pull_request.body }}" > file.txt`
- **Good:** Use env block and reference variables:
  ```yaml
  env:
    PR_BODY: ${{ github.event.pull_request.body }}
  run: |
    printf '%s' "$PR_BODY" > file.txt
  ```

## Architecture & Patterns

### Project Structure
```
src/aap_eda/
├── api/                     # REST API layer (views, serializers, filters)
├── core/                    # Core business logic (models, migrations, enums)
│   ├── models/              # Django ORM models
│   ├── management/          # Django management commands
│   ├── migrations/          # Database migrations
│   ├── tasking/             # Task/job scheduling (dispatcherd)
│   └── utils/               # Helper utilities
├── services/                # Business logic services
│   ├── activation/          # Activation lifecycle management
│   │   └── engine/          # Execution engines (Kubernetes, Podman)
│   └── project/             # Project management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ansible/eda-server](https://github.com/ansible/eda-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
