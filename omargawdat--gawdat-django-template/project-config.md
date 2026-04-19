---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Django REST API template project with a modular app structure and comprehensive CI/CD pipeline. The project
uses Docker for local development and includes payment systems and location services.

## Code Style Guidelines

### Core Principles

**SEEK**: Smart, Simple, Best Practice, Minimum Code, Maximum Effect

- ✅ **Smart solutions** - Think before coding, use Django's built-in features
- ✅ **Follow best practices** - Use Django patterns, DRF conventions, PEP 8
- ✅ **Keep it simple** - Prefer readable code over clever code
- ✅ **Minimum code** - Don't write what Django already provides
- ✅ **Maximum effect** - Each line should serve a clear purpose

**Examples:**

```python
# ❌ Bad - Reinventing the wheel
def get_user_by_email(email):
    try:
        return User.objects.get(email=email)
    except User.DoesNotExist:
        return None

# ✅ Good - Use Django's built-in
def get_user_by_email(email):
    return User.objects.filter(email=email).first()

# ❌ Bad - Over-engineering
class CustomerService:
    def __init__(self, customer_repository):
        self.repository = customer_repository

    def get_customer(self, id):
        return self.repository.find_by_id(id)

# ✅ Good - Simple and direct
def get_customer(customer_id: int) -> Customer:
    return get_object_or_404(Customer, pk=customer_id)
```

### Documentation

- **Keep method docstrings concise** - Aim for 1-3 lines maximum
- Focus on **what** the method does, not **how** (the code shows how)
- Only add detailed docstrings for complex business logic or non-obvious behavior
- Use inline comments for tricky code sections instead of lengthy docstrings

**Example - Good (Concise):**
```python
def get_customer(customer_id: int) -> Customer:
    """Get customer by ID or raise 404."""
    return get_object_or_404(Customer, pk=customer_id)
```

**Example - Avoid (Too verbose):**
```python
def get_customer(customer_id: int) -> Customer:
    """
    Retrieve a customer instance from the database.

    Args:
        customer_id: The primary key of the customer to retrieve

    Returns:
        Customer: The customer instance matching the provided ID

    Raises:
        Http404: If no customer exists with the provided ID
    """
    return get_object_or_404(Customer, pk=customer_id)
```

## Package Management

This project uses **uv** package manager with dependencies defined in `pyproject.toml`:

- **pyproject.toml** - Single source of truth for all dependencies
  - `[project.dependencies]` - Core/base dependencies (Django, DRF, etc.)
  - `[project.optional-dependencies]` - Production extras (gunicorn, sentry, S3)
  - `[dependency-groups]` - PEP 735 groups (dev, test, typing, lint)
- **uv.lock** - Lockfile with exact versions and SHA256 hashes (auto-generated)

### Managing Dependencies

```bash
# Add a new dependency
uv add package-name

# Add to dev group
uv add --group dev package-name

# Remove a dependency
uv remove package-name

# Update specific package
uv lock --upgrade-package django

# Update all packages
uv lock --upgrade
```

## Development Commands

### Local Development Setup

```bash
# Copy environment variables
cp dummy.env .env

# Start development environment with Docker Compose
docker-compose -f docker-compose.local.yml up --build -d

# Run Django management commands
docker compose -f docker-compose.local.yml run --rm django python manage.py [command]

# Examples:
docker compose -f docker-compose.local.yml run --rm django python manage.py makemigrations
docker compose -f docker-compose.local.yml run --rm django python manage.py migrate
docker compose -f docker-compose.local.yml run --rm django python manage.py createsuperuser
```

### Code Generation Commands

This project includes a custom management command to generate admin dashboard boilerplate code:

#### Generate Dashboard (Admin Interface)

```bash
# Generate all admin dashboard components for a model
docker compose -f docker-compose.local.yml run --rm django python manage.py generate_dashboard <app_name> <model_name>

# Example: Generate dashboard for Customer model
docker compose -f docker-compose.local.yml run --rm django python manage.py generate_dashboard users Customer

# Options:
# --skip-inline    Skip generating inline admin
```

**What it generates:**
- `admin/<model_name>/list_view.py` - List view configuration (columns, filters, search)
- `admin/<model_name>/change_view.py` - Change/detail view configuration
- `admin/<model_name>/permissions.py` - Permission handling (uses field names directly)
- `admin/<model_name>/display.py` - Custom display methods
- `admin/<model_name>/resource.py` - Export/import configuration
- `admin/<model_name>/inline.py` - Inline admin for related models
- `admin/<model_name>/admin.py` - Main admin registration
- `admin/<model_name>/CHECKLIST.md` - Configuration review checklist

**Usage workflow:**
1. Create your model in `models/` directory
2. Run migrations
3. Run `generate_dashboard` command
4. Review and customize generated files
5. Follow the CHECKLIST.md for configuration review


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/omargawdat) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
