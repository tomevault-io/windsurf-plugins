---
trigger: always_on
description: > Guidelines for AI coding agents operating in this repository.
---

# AGENTS.md

> Guidelines for AI coding agents operating in this repository.

## Project Overview

Django library providing database-enforced multitenancy using PostgreSQL Row-Level Security (RLS).
Python 3.11+, Django 4.2+, PostgreSQL 15+. Uses `uv` for dependency management and `hatchling` as build backend.

## Build & Dependencies

```bash
uv sync --group dev --group test   # install all deps (dev + test)
uv run pre-commit install          # install pre-commit hooks
```

## Lint / Format / Type-Check

```bash
uv run ruff check .                # lint (all rules in pyproject.toml)
uv run ruff check --fix .          # lint with auto-fix
uv run ruff format .               # format (Black-compatible)
uv run ruff format --check .       # format check only (CI mode)
uv run mypy django_rls_tenants     # strict type-check (production code only)
```

Pre-commit hooks run ruff check, ruff format, and mypy automatically on every commit.

## Testing

Tests require a running PostgreSQL instance. Configure via env vars
(`POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_HOST`, `POSTGRES_PORT`)
or use defaults from `tests/settings.py` (db: `django_rls_tenants_test`, user/pass: `postgres`).

```bash
uv run pytest                              # run all tests
uv run pytest tests/test_rls/test_guc.py   # run a single test file
uv run pytest tests/test_rls/test_guc.py::test_set_get_roundtrip  # single test
uv run pytest -k "guc and not local"       # keyword filter
uv run pytest -m integration               # only integration tests (need live PG)
uv run pytest --cov                        # with coverage (fail_under = 90%)
```

### Test Organization

```
tests/
├── conftest.py              # shared fixtures
├── settings.py              # Django settings for test suite
├── test_rls/                # unit tests for rls/ layer
├── test_tenants/            # unit tests for tenants/ layer
├── test_integration/        # end-to-end tests (marked @pytest.mark.integration)
└── test_layering.py         # enforces rls/ ← tenants/ import boundary
```

## Architecture

The library has two internal layers with a strict import boundary:

- **`django_rls_tenants/rls/`** — Generic PostgreSQL RLS primitives (GUC helpers,
  `RLSConstraint`, `RLSM2MConstraint`, context managers). **Zero imports from `tenants/`.**
- **`django_rls_tenants/tenants/`** — Django multitenancy built on `rls/` (models,
  managers, middleware, config, testing utilities).
- **`django_rls_tenants/operations.py`** — Migration operations (`AddM2MRLSPolicy`)
  for M2M join table RLS policies. Imports from `rls/` for shared SQL builders and validation.

This boundary is enforced by `tests/test_layering.py`. Never import from `tenants/` in `rls/`.

### M2M RLS Support

M2M through tables get `EXISTS`-based subquery RLS policies. Three components
cooperate:

- `RLSM2MConstraint` (in `rls/constraints.py`) — constraint for `Meta.constraints`
- `AddM2MRLSPolicy` (in `operations.py`) — reversible migration operation
- `register_m2m_rls()` (in `tenants/models.py`) — auto-detection in `AppConfig.ready()`

All three share SQL builder functions (`_build_m2m_conditions`,
`_build_m2m_create_sql`, `_build_m2m_drop_sql`) from `rls/constraints.py` to
avoid duplication. All SQL-interpolated inputs are validated via
`_validate_field_name`, `_validate_model_path`, `_validate_pk_type`, and
`_validate_guc_name_for_ddl`.

### Multi-Database Support

The middleware sets GUC variables on all database aliases listed in
`RLS_TENANTS["DATABASES"]` (default: `["default"]`). A `connection_created`
signal handler in `apps.py` also sets GUCs on lazily-created connections.
The safety-net `request_finished` handler clears GUCs on all configured aliases.

When modifying middleware or GUC-related code, ensure all database iteration
uses `conf.DATABASES` rather than hardcoding `"default"`.

### Strict Mode

When `STRICT_MODE=True`, `TenantQuerySet` evaluation methods raise
`NoTenantContextError` if no RLS context is active. A second `ContextVar`
(`_rls_context_active` in `state.py`) tracks whether `tenant_context()`,
`admin_context()`, `RLSTenantMiddleware`, or `for_user()` has established
context. This distinguishes "no context" (should raise) from "admin context"
(both have `_current_tenant_id=None`). All entry points set the flag to `True`
on entry and restore via token on exit. The guard is in
`TenantQuerySet._check_strict_mode()`.

## Code Style

### General Formatting

- **Line length:** 99 characters.
- **Formatter:** ruff format (Black-compatible). Double quotes for strings.
- **Trailing commas** in all multi-line constructs (enforced by formatter).
- **Two blank lines** between top-level definitions; one blank line between methods.

### Imports

- **Every `.py` file** with content must start with a module docstring followed by
  `from __future__ import annotations`.
- **Import order** (enforced by ruff isort): stdlib → third-party → first-party.
  Groups separated by a blank line. First-party package: `django_rls_tenants`.
- **Absolute imports only.** No relative imports.
- **`from X import Y`** preferred over bare `import X` for specific symbols.
- Combine multiple imports from the same module on one line.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dvoraj75/django-rls-tenants](https://github.com/dvoraj75/django-rls-tenants) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
