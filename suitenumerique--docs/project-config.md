---
trigger: always_on
description: This file contains guidelines for AI agents coding in this repository.
---

# Agent Guidelines for Docs

This file contains guidelines for AI agents coding in this repository.

## Project Overview

**La Suite Docs** is a collaborative text editor built by DINUM (French government) and ZenDiS (German government). It features real-time editing, offline support, AI actions, and multi-format export (PDF/DOCX/ODT).

Repository: https://github.com/suitenumerique/docs

## Monorepo Structure

- `src/backend/` - Django REST API (Python 3.14+)
- `src/frontend/apps/impress/` - Main Next.js 15 application (TypeScript)
- `src/frontend/apps/e2e/` - Playwright E2E tests
- `src/frontend/packages/i18n/` - Shared i18n utilities
- `src/frontend/packages/eslint-plugin-docs/` - Custom ESLint plugin
- `src/frontend/servers/y-provider/` - Conversion service only (Express, `POST /api/convert/`). It no longer serves any websocket
- `src/yhub-server/` - Collaboration server: a thin TypeScript wrapper around `@y/hub` (websockets, REST routes, worker). It holds the content of the documents
- `src/mail/` - Email templates (MJML)
- `src/helm/` - Kubernetes/Helm deployment

## Development Commands

### Setup and Services

```bash
make bootstrap          # Full dev setup (build + migrate + demo + run)
make build              # Build all Docker containers
make run                # Start all services
make stop               # Stop services
make build-yhub         # Build the collaboration server (yhub) image
make migrate-yhub       # Create/upgrade the yhub database schema (`yarn init-db`), safe to re-run
make status             # Check running services
```

### Backend (Python/Django)

Tests run inside Docker containers. You must first build the backend image and ensure the `lasuite-network` Docker network exists before running tests.

```bash
make build-backend                     # Build the backend Docker image (required before first test run)
docker network create lasuite-network  # Create the external network (required once)
bin/pytest -n auto                     # Run all backend tests in parallel
bin/pytest -n auto path/to/test        # Run specific test file/directory
bin/pytest -n auto path/to/test.py::TestClass::test_method  # Run single test via docker compose
make lint                              # ruff format + ruff check + pylint
make lint-ruff-format                  # Format only
make lint-ruff-check                   # Lint only
make migrate                           # Run database migrations
make makemigrations                    # Create new migrations
make resetdb                           # Flush DB + create superuser (admin/admin)
```

### Frontend (TypeScript/Next.js)

```bash
# From src/frontend/apps/impress/:
yarn dev                # Development server (port 3000)
yarn build              # Production build (includes prettier + stylelint checks)
yarn lint               # TypeScript check + ESLint
yarn test               # Run Vitest tests
yarn prettier           # Format code
yarn stylelint          # Lint CSS

# From project root:
make frontend-lint      # Lint all frontend workspaces
make frontend-test      # Run frontend tests
```

### Collaboration server (yhub, TypeScript)

```bash
# From src/yhub-server/ (standalone package: its own package.json and yarn.lock,
# not a workspace of src/frontend):
yarn test               # Vitest unit tests (@y/hub is mocked, no store needed)
yarn typecheck          # tsc --noEmit
yarn lint               # ESLint
yarn build              # Compile to dist/
yarn init-db            # Create/upgrade the yhub postgres schema (what `make migrate-yhub` runs)
```

The container starts with `node --import ./dist/sentry.js dist/server.js`: keep the
`--import` when overriding the command. `src/yhub-server/README.md` is the reference
for everything this server does (permissions, roles, migration, storage, metrics).

### Mails

```bash
make mails-install      # Install dependencies
make mails-build        # Convert MJML to HTML + plaintext
```

### Helm/Kubernetes Deployment

```bash
make build-k8s-cluster       # Create local Kind cluster
make start-tilt             # Start Tilt for hot-reload
# From src/helm/: helmfile -n impress -e dev apply/destroy/diff
```

### Dev Service URLs

- Frontend: http://localhost:3000
- Backend API/Admin: http://localhost:8071
- Keycloak (auth): http://localhost:8083
- MinIO (S3): http://localhost:9000
- Collaboration server (yhub): http://localhost:3002 (websocket on `/collaboration/ws/v1/docs/{docid}`)
- yhub PostgreSQL: localhost:5433 (its own database, apart from the backend's on 15432)
- Mailcatcher: http://localhost:1081

## Architecture

### Backend

- **Framework**: Django + DRF, configured via `django-configurations` (`src/backend/impress/settings.py`)
- **Main app**: `src/backend/core/` (models, API viewsets, services, authentication)
- **API**: REST on `/api/v1.0/` with nested routes (e.g., `/documents/{id}/accesses/`)
- **Auth**: OIDC via `mozilla-django-oidc` (Keycloak in dev)
- **Background tasks**: Celery + Redis
- **Database**: PostgreSQL 16 with `django-treebeard` for document hierarchy

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [suitenumerique/docs](https://github.com/suitenumerique/docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
