---
trigger: always_on
description: This file provides instructions for GitHub Copilot when working with the ArtificialU codebase.
---

# GitHub Copilot Instructions for ArtificialU

This file provides instructions for GitHub Copilot when working with the ArtificialU codebase.

## Project Overview

ArtificialU is an AI-powered educational content platform that generates university lectures with distinct professor personalities and converts them to audio using text-to-speech.

### Technology Stack

**Backend:**

- Python 3.13+ with FastAPI
- PostgreSQL with SQLAlchemy ORM
- Hatch for environment management
- AI: Anthropic Claude, Google Gemini, OpenAI
- TTS: ElevenLabs
- Storage: MinIO (dev) / S3 (prod)
- Background jobs: Custom async worker with PostgreSQL-backed queue

**Frontend:**

- SolidJS with TypeScript
- TailwindCSS v4
- Auth0 for authentication
- Kobalte UI for components
- Vite for build tooling

## Architecture

### Three-Tier Backend Architecture

1. **API Layer** (`artificial_u/api/routers/`)
   - FastAPI routers define REST endpoints
   - Handle HTTP concerns (request/response, validation)
   - Delegate business logic to service layer

2. **Service Layer**
   - **API Services** (`artificial_u/api/services/`): HTTP-aware, coordinate multiple core services
   - **Core Services** (`artificial_u/services/`): Pure domain logic, no HTTP dependencies
   - **Generator Services**: AI content generation workflows

3. **Repository Layer** (`artificial_u/models/`)
   - SQLAlchemy models and repository pattern
   - Repository factory for dependency injection
   - All database access goes through repositories

### Key Directory Structure

```
artificial_u/
├── api/                 # FastAPI application
│   ├── app.py          # Application factory
│   ├── dependencies.py # Dependency injection
│   ├── routers/        # API endpoints
│   ├── services/       # API-layer services
│   ├── models/         # Pydantic request/response models
│   ├── middlewares/    # CORS, logging, error handling
│   └── security/       # Auth0 JWT validation
├── models/             # SQLAlchemy models & repositories
├── services/           # Core business logic
├── audio/              # TTS processing
├── integrations/       # External API clients
├── prompts/            # AI prompt templates
└── config/             # Configuration management

web/src/
├── api/                # API client & service calls
├── auth/               # Auth0 integration
├── components/         # Reusable UI components
├── pages/              # Route page components
└── utils/              # Utilities (theme, SSE, etc.)
```

## Development Commands

### Python Backend

All Python commands use `hatch` for environment management:

```bash
# Run CLI commands
hatch run artificial_u --help
hatch run artificial_u list-courses

# Testing
hatch run pytest                    # All tests
hatch run pytest -m unit            # Unit tests only
hatch run pytest -m integration     # Integration tests only
hatch run pytest --cov=artificial_u # Coverage report

# Code quality
hatch run black artificial_u        # Format code
hatch run isort artificial_u        # Sort imports
hatch run flake8 artificial_u       # Lint
hatch run mypy artificial_u         # Type check

# Database
hatch run python scripts/initialize_db.py       # Setup dev database
hatch run python scripts/setup_test_db.py      # Setup test database
hatch run python scripts/run_alembic.py upgrade head  # Run migrations

# API server
hatch run uvicorn artificial_u.api.app:app --reload --host 0.0.0.0 --port 8000

# Background worker
hatch run python -m artificial_u.api.worker
```

### Frontend (SolidJS)

All frontend commands run from the `web/` directory using `pnpm`:

```bash
cd web

pnpm dev              # Start dev server (http://localhost:5173)
pnpm build            # Production build
pnpm preview          # Preview production build

pnpm lint             # ESLint
pnpm lint:fix         # Auto-fix ESLint issues
pnpm format           # Format with BiomeJS

pnpm test             # Run tests
pnpm test:watch       # Watch mode
pnpm test:coverage    # Coverage report
```

### Docker & Services

```bash
docker compose up -d     # Start postgres, minio
docker compose down      # Stop services
docker compose logs -f   # View logs

# Makefile shortcuts
make dev-setup          # Complete setup (services + database)
make check              # Run linting + tests
make test               # Run all tests
make lint               # All linting checks
make format             # Format code
make run-api            # Start FastAPI server
```

## Code Quality Standards

### Python

- **Line length**: 100 characters (enforced by black, isort, flake8)
- **Formatting**: Use Black formatter with isort for imports
- **Type hints**: Encouraged and checked with mypy on `artificial_u/` directory
- **Import order**: Use isort profile "black" for consistency
- **Linting**: Must pass flake8 checks
- **Pre-commit hooks**: Run black, isort, flake8 automatically

### TypeScript

- **Linting**: ESLint with TypeScript parser
- **Formatting**: BiomeJS for code formatting
- **Styling**: Stylelint for CSS
- **Type coverage**: Full TypeScript coverage expected
- **Components**: Use Kobalte UI primitives
- **State**: SolidJS signals for reactivity
- **Styling**: TailwindCSS v4 utilities

## Testing Guidelines


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ballPointPenguin/artificial-u](https://github.com/ballPointPenguin/artificial-u) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
