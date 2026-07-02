---
trigger: always_on
description: Sentry is a developer-first error tracking and performance monitoring platform. This repository contains the main Sentry application, which is a large-scale Django application with a React frontend.
---

# Sentry Development Guide for Claude

## Overview

Sentry is a developer-first error tracking and performance monitoring platform. This repository contains the main Sentry application, which is a large-scale Django application with a React frontend.

## Tech Stack

### Backend

- **Language**: Python 3.13+
- **Framework**: Django 5.2+
- **API**: Django REST Framework with drf-spectacular for OpenAPI docs
- **Task Queue**: Celery 5.5+
- **Databases**: PostgreSQL (primary), Redis, ClickHouse (via Snuba)
- **Message Queue**: Kafka, RabbitMQ
- **Cloud Services**: Google Cloud Platform (Bigtable, Pub/Sub, Storage, KMS)

### Frontend

- **Language**: TypeScript
- **Framework**: React 19
- **Build Tool**: Rspack (Webpack alternative)
- **State Management**: Reflux, React Query (TanStack Query)
- **Styling**: Emotion (CSS-in-JS), Less
- **Testing**: Jest, React Testing Library

### Infrastructure

- **Container**: Docker (via devservices)
- **Package Management**: pnpm (Node.js), pip (Python)
- **Node Version**: 22 (managed by Volta)

## Project Structure

```
sentry/
├── src/
│   ├── sentry/           # Main Django application
│   │   ├── api/          # REST API endpoints
│   │   ├── models/       # Django models
│   │   ├── tasks/        # Celery tasks
│   │   ├── integrations/ # Third-party integrations
│   │   ├── issues/       # Issue tracking logic
│   │   └── web/          # Web views and middleware
│   ├── sentry_plugins/   # Plugin system
│   └── social_auth/      # Social authentication
├── static/
│   ├── app/              # React application
│   │   ├── components/   # Reusable React components
│   │   ├── views/        # Page components
│   │   ├── stores/       # State management
│   │   └── utils/        # Utility functions
│   └── fonts/            # Font files
├── tests/                # Test suite
├── fixtures/             # Test fixtures
├── devenv/               # Development environment config
├── migrations/           # Database migrations
└── config/               # Configuration files
```

## Key Commands

### Development Setup

```bash
# Install dependencies and setup development environment
make develop

# Or use the newer devenv command
devenv sync

# Start the development server
pnpm run dev

# Start only the UI development server with hot reload
pnpm run dev-ui
```

### Testing

```bash
# Run Python tests
pytest

# Run JavaScript tests
pnpm test

# Run specific test file
pytest tests/sentry/api/test_base.py
pnpm test components/avatar.spec.tsx
```

### Code Quality

```bash
# Preferred: Run pre-commit hooks on specific files
pre-commit run --files src/sentry/path/to/file.py

# Run all pre-commit hooks
pre-commit run --all-files

# Individual linting tools (use pre-commit instead when possible)
black --check  # Run black first
isort --check
flake8

# JavaScript/TypeScript linting
pnpm run lint:js

# Fix linting issues
pnpm run fix
```

### Database Operations

```bash
# Run migrations
sentry django migrate

# Create new migration
sentry django makemigrations

# Reset database
make reset-db
```

## Development Services

Sentry uses `devservices` to manage local development dependencies:

- **PostgreSQL**: Primary database
- **Redis**: Caching and queuing
- **Snuba**: ClickHouse-based event storage
- **Relay**: Event ingestion service
- **Symbolicator**: Debug symbol processing
- **Taskbroker**: Asynchronous task processing
- **Spotlight**: Local debugging tool

## AI Assistant Quick Decision Trees

### "User wants to add an API endpoint"

1. Check if endpoint already exists: `grep -r "endpoint_name" src/sentry/api/`
2. Inherit from appropriate base:
   - Organization-scoped: `OrganizationEndpoint`
   - Project-scoped: `ProjectEndpoint`
   - Region silo: `RegionSiloEndpoint`
3. File locations:
   - Endpoint: `src/sentry/api/endpoints/{resource}.py`
   - URL: `src/sentry/api/urls.py`
   - Test: `tests/sentry/api/endpoints/test_{resource}.py`
   - Serializer: `src/sentry/api/serializers/models/{model}.py`

### "User wants to modify frontend component"

1. Component location: `static/app/components/` (reusable) or `static/app/views/` (page-specific)
2. ALWAYS use TypeScript
3. ALWAYS write test in same directory with `.spec.tsx`
4. Style with Emotion, NOT inline styles or CSS files
5. State: Use hooks (`useState`), NOT Reflux for new code

### "User wants to add a Celery task"

1. Location: `src/sentry/tasks/{category}.py`
2. Use `@instrumented_task` decorator
3. Set appropriate `queue` and `max_retries`
4. Test location: `tests/sentry/tasks/test_{category}.py`

## Critical Patterns (Copy-Paste Ready)

### API Endpoint Pattern

```python
# src/sentry/api/endpoints/organization_details.py
from rest_framework.request import Request
from rest_framework.response import Response
from sentry.api.base import region_silo_endpoint
from sentry.api.bases.organization import OrganizationEndpoint
from sentry.api.serializers import serialize
from sentry.api.serializers.models.organization import DetailedOrganizationSerializer

@region_silo_endpoint
class OrganizationDetailsEndpoint(OrganizationEndpoint):
    publish_status = {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [code-review-benchmark/sentry__sentry__copilot-v2__PR93824__20260624](https://github.com/code-review-benchmark/sentry__sentry__copilot-v2__PR93824__20260624) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-02 -->
