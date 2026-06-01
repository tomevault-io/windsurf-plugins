---
trigger: always_on
description: Apply when navigating or understanding the codebase structure
---


# Codebase Directory Structure

This document provides detailed directory trees for the main components of the Rhesis platform.

---

## Backend (`apps/backend/`)

FastAPI-based REST API with Celery task processing.

```
apps/backend/
├── pyproject.toml              # Python package configuration and dependencies (uv)
├── uv.lock                     # Locked dependency versions
├── Makefile                    # Build automation: test, lint, format
├── Dockerfile                  # Production container image
├── Dockerfile.dev              # Development container with hot-reload
├── start.sh                    # Container entrypoint script
├── migrate.sh                  # Database migration runner
├── CHANGELOG.md                # Version history and release notes
├── CONTRIBUTING.md             # Development guidelines
├── README.md                   # Project overview and setup
│
└── src/rhesis/backend/         # Main source code
    │
    ├── app/                    # FastAPI application core
    │   ├── main.py             # FastAPI entry point, router registration
    │   ├── database.py         # SQLAlchemy engine, session management
    │   ├── dependencies.py     # Dependency injection (auth, db sessions)
    │   ├── crud.py             # Generic CRUD operations base
    │   ├── constants.py        # Application-wide constants
    │   ├── error_handlers.py   # Global exception handlers
    │   │
    │   ├── auth/               # Authentication & authorization
    │   │   ├── oauth.py        # OAuth2 flow implementation
    │   │   ├── permissions.py  # Role-based permission checks
    │   │   ├── decorators.py   # Auth decorators for routes
    │   │   ├── token_utils.py  # JWT token generation/parsing
    │   │   ├── token_validation.py  # Token verification logic
    │   │   ├── user_utils.py   # User lookup helpers
    │   │   ├── auth_utils.py   # General auth utilities
    │   │   └── url_utils.py    # Auth URL builders
    │   │
    │   ├── config/             # Application configuration
    │   │   └── cascade_config.py  # Cascade delete/update rules
    │   │
    │   ├── middleware/         # FastAPI middleware
    │   │   └── organization_filter.py  # Multi-tenant org filtering
    │   │
    │   ├── models/             # SQLAlchemy ORM models (39 files)
    │   │   ├── base.py         # Base model with common fields
    │   │   ├── mixins.py       # Reusable mixins (timestamps, soft delete)
    │   │   ├── user.py, organization.py, project.py
    │   │   ├── test.py, test_set.py, test_run.py, test_result.py
    │   │   ├── endpoint.py, prompt.py, metric.py
    │   │   └── ...             # Other domain models
    │   │
    │   ├── schemas/            # Pydantic request/response schemas (43 files)
    │   │   ├── base.py         # Base schema with common fields
    │   │   └── ...             # Corresponding schemas for each model
    │   │
    │   ├── routers/            # FastAPI route handlers (40 files)
    │   │   ├── base.py         # Base router with common CRUD endpoints
    │   │   ├── auth.py         # Authentication endpoints
    │   │   ├── test.py, test_run.py, telemetry.py
    │   │   └── ...             # Other resource routers
    │   │
    │   ├── services/           # Business logic layer (87 files)
    │   │   ├── test_execution.py   # Test execution orchestration
    │   │   ├── test_run.py         # Test run management
    │   │   ├── organization.py     # Org provisioning
    │   │   ├── task_management.py  # Celery task management
    │   │   │
    │   │   ├── connector/      # External system connectors
    │   │   │   ├── manager.py, handler.py
    │   │   │   ├── handlers/   # Connector implementations
    │   │   │   └── mapping/    # Data mapping between systems
    │   │   │
    │   │   ├── endpoint/       # Endpoint testing services
    │   │   │   ├── service.py  # Endpoint validation and testing
    │   │   │   └── validation.py
    │   │   │
    │   │   ├── invokers/       # Target system invocation
    │   │   │   ├── base.py     # Base invoker interface
    │   │   │   ├── rest_invoker.py, sdk_invoker.py, websocket_invoker.py
    │   │   │   ├── auth/       # Invoker authentication
    │   │   │   ├── conversation/  # Multi-turn handling
    │   │   │   └── templating/    # Request/response templating
    │   │   │
    │   │   ├── stats/          # Statistics calculation
    │   │   │   └── calculator.py, test_result.py
    │   │   │
    │   │   ├── telemetry/      # Telemetry processing
    │   │   │   ├── enricher.py, tree_builder.py, linking_service.py
    │   │   │
    │   │   └── handlers/       # Document handlers
    │   │
    │   ├── templates/          # Jinja2 prompt templates (*.jinja2)
    │   │
    │   ├── utils/              # Utility functions (18 files)
    │   │   ├── crud_utils.py, odata.py, encryption.py, rate_limit.py
    │   │
    │
    ├── alembic/                # Database migrations
    │   ├── env.py              # Alembic environment config
    │   ├── versions/           # Migration scripts (115+ files)
    │   ├── templates/          # SQL templates for migrations
    │   └── utils/              # Migration utilities
    │
    ├── tasks/                  # Celery background tasks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rhesis-ai/rhesis](https://github.com/rhesis-ai/rhesis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
