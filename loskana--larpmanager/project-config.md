---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**LarpManager** is a Django-based web application for managing LARP (Live Action Role-Playing) events. It provides comprehensive functionality for event organization, character management, registrations, accounting, and more.

## Documentation

- **[Features and Permissions Guide](docs/01-features-and-permissions.md)** - Comprehensive guide for creating features, views, and permissions
- **[Roles and Context Guide](docs/02-roles-and-context.md)** - How to structure views with context and understand role-based permissions
- **[Configuration System Guide](docs/03-configuration-system.md)** - How to add customizable settings without modifying models
- **[Localization Guide](docs/04-localization.md)** - How to write translatable code and manage translations
- **[Playwright Testing Guide](docs/05-playwright-testing.md)** - How to write and run end-to-end tests
- **[Feature Descriptions](docs/06-feature-descriptions.md)** - Complete reference of all available features
- **[Test Database Schema Versioning](docs/07-test-database-schema-versioning.md)** - How the automatic schema version detection works
- **[Security Best Practices](docs/08-security-best-practices.md)** - Critical security requirements including UUID usage
- **[README.md](README.md)** - Installation, deployment, and contribution guidelines

## Package Management

The project uses **uv** for fast and reliable Python package management. All dependencies are defined in `pyproject.toml`.

### Installing uv
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### Installing dependencies
```bash
# System-wide (for Docker and CI)
uv pip install --system -r pyproject.toml

# In a virtual environment
uv venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
uv pip install -r pyproject.toml
```

### Upgrading dependencies
Use the `./scripts/pip_upgrade.sh` script which has been updated for uv:
```bash
./scripts/pip_upgrade.sh
```

## Virtual Environment

The virtual environment is located at `.venv` in the project working directory. Always use it directly for Python and ruff commands:
```bash
.venv/bin/ruff check <file>
.venv/bin/ruff format <file>
.venv/bin/python manage.py <command>
```
Do not prefix with `source .venv/bin/activate &&`, invoke the venv binaries directly (e.g. `.venv/bin/ruff`, `.venv/bin/python`).
For pytest and scripts, use `source <venv>/bin/activate` or `./scripts/test.sh`.

## Development Commands

### Common Development Tasks
- **Run all tests**: `./scripts/test.sh [workers]` (default: 6 workers)
- **Run specific test**: `pytest larpmanager/tests/specific_test.py`
- **Run unit tests only**: `./scripts/test_unit.sh`
- **Run playwright tests only**: `./scripts/test_playwright.sh`
- **Create migrations**: `python manage.py makemigrations`
- **Apply migrations**: `python manage.py migrate`
- **Load test fixtures**: `python manage.py reset` (creates test org with users: `admin`, `orga@test.it`, `user@test.it` - password: `banana`) - automatically runs makemigrations and migrate
- **Create superuser**: `python manage.py createsuperuser`
- **Run automation tasks**: `python manage.py automate` (should be scheduled daily, handles advanced features)
- **Lint code**: `ruff check`
- **Format code**: `ruff format`
- **Translation updates**: `./scripts/translate.sh` (requires `DEEPL_API_KEY` in dev settings)
- **Record playwright tests**: `./scripts/record-test.sh`
- **Update test dump**: `python manage.py dump_test` (required after model/fixture changes) - automatically runs makemigrations, migrate, and reset; auto-adds schema version marker

### Feature Management
- **Export features to fixtures**: `python manage.py export_features` (run before pushing new features)
- **Import features from fixtures**: `python manage.py import_features` (automatically run during deploy)

### Frontend Development
- **Install frontend dependencies**: `cd larpmanager/static && npm install`
- **Frontend dependencies are in**: `larpmanager/static/package.json`

### Docker Development
- **Build and run**: `docker compose up --build`
- **Create superuser in container**: `docker exec -it larpmanager python manage.py createsuperuser`
- **Deploy updates**: `docker exec -it larpmanager scripts/deploy.sh` (graceful restart with migrations)
- **Build CI image**: `./scripts/build_ci_image.sh` (for updating CI Docker image)
- **Build and push CI image**: `./scripts/build_ci_image.sh --push` (requires GHCR authentication)

## Architecture Overview

### Django App Structure
- **Main Django project**: `main/` - Contains settings, URLs, WSGI/ASGI configuration
- **Core app**: `larpmanager/` - Contains all models, views, and business logic
- **Settings structure**: `main/settings/` with environment-specific configs (dev, prod, test, ci)

### Key Model Categories
Models are organized in `larpmanager/models/` by domain:
- **Organizations & Events**: `association.py`, `event.py` - Association, Event, Run management
- **User Management**: `member.py` - Custom Member model, character creation and management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LoSkana/larpmanager](https://github.com/LoSkana/larpmanager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
