---
trigger: always_on
description: This file provides essential information for coding agents working with the QueryWeaver repository. Follow these instructions to work efficiently and avoid common pitfalls.
---

# QueryWeaver Copilot Instructions

This file provides essential information for coding agents working with the QueryWeaver repository. Follow these instructions to work efficiently and avoid common pitfalls.

## Repository Overview

QueryWeaver is an open-source Text2SQL tool that transforms natural language into SQL using graph-powered schema understanding. Built with Python/FastAPI and FalkorDB (graph database), it provides a web interface for natural language database queries with OAuth authentication.

**Key Technologies:**
- **Backend**: Python 3.12+, FastAPI 0.115.0+, FalkorDB (Redis-based graph database)
- **AI/ML**: LiteLLM with Azure OpenAI/OpenAI integration for text-to-SQL generation
- **Testing**: pytest for unit tests, Playwright for E2E testing
- **Dependencies**: uv for package management
- **Authentication**: authlib with Google/GitHub OAuth
- **Deployment**: Docker support, Vercel configuration

**Repository Size**: ~50 Python files, medium complexity web application with comprehensive test suite.

## Essential Build & Validation Commands

Follow this order for a reliable local setup; if you customize the steps, ensure each prerequisite (dependencies, `.env`, Playwright) is completed.

### 1. Initial Setup (recommended for new contributors)
```bash
# Install uv if not available
pip install uv
# or visit https://docs.astral.sh/uv/getting-started/installation/

# Install dependencies (backend + frontend) and prepare dev tools
# Recommended: use the Make helper which installs Python deps and frontend deps
make install

# Prepare the full development environment (installs Playwright browsers too)
# This runs `make install` then Playwright install steps.
make setup-dev

# OR manual steps if you prefer more granular control:
# uv sync
# uv run playwright install chromium
# uv run playwright install-deps

# Set up environment file
cp .env.example .env
# Edit .env with required values (see Environment Setup section)
```

Note: This project includes a TypeScript frontend in `app/` that must be built before a production run. Node.js and npm are required for the frontend build; `make install` will attempt to install frontend deps in `app/` when present. After `make install`, run `make build-prod` (or `cd app && npm run build`) to compile the TypeScript into the static bundle (build output: `app/public/js/app.js`).

### 2. Development Environment Setup
```bash
# Complete development setup (includes Playwright browsers)
make setup-dev

# OR manual steps:
uv sync
uv run playwright install chromium
uv run playwright install-deps
```

### 3. Testing Commands
```bash
# IMPORTANT: Unit tests require FalkorDB running or will fail with connection errors
# You can start a local test FalkorDB using the included Make helper
make docker-falkordb

# Run unit tests only (safer, doesn't require browser)
make test-unit

# Run E2E tests (requires Playwright setup)
make test-e2e

# Run E2E tests with visible browser (for debugging)
make test-e2e-headed

# Run all tests
make test

# Stop test database when done
make docker-stop
```

### 4. Linting & Code Quality
```bash
# Run pylint (can be run without FalkorDB)
make lint
# OR manually: uv run pylint $(git ls-files '*.py')
```

### 5. Running the Application

```bash
# Development server with debug mode
make run-dev
# OR manually: uv run uvicorn api.index:app --host "localhost" --port "5000" --reload

# Production mode
make run-prod
# OR manually: uv run uvicorn api.index:app --host "localhost" --port "5000"
```

Important: If you're preparing a production deployment or have changed frontend code, run `make build-prod` (or `make build-dev` for a development build) first to produce the static bundle used by the app.

### 5a. Running with Docker

You can run QueryWeaver using Docker without installing Python dependencies locally:

```bash
docker run -p 5000:5000 -it falkordb/queryweaver
```

#### Passing Environment Variables

You can pass environment variables individually using `-e` flags, or provide a full environment file using `--env-file`:

```bash
docker run -p 5000:5000 --env-file .env falkordb/queryweaver
```

Use the provided `.env.example` as a template:

```bash
cp .env.example .env
# Edit .env with your values, then run:
docker run -p 5000:5000 --env-file .env falkordb/queryweaver
```

### 6. Cleanup
```bash
# Clean test artifacts
make clean
```

## Environment Setup Requirements

Create `.env` file from `.env.example` and configure these essential variables:

```bash
# REQUIRED for FastAPI to start
FASTAPI_SECRET_KEY=your_super_secret_key_here

# Optional: set application environment (development, staging, production)
# Default: development (affects session cookie security for OAuth)
APP_ENV=development

# REQUIRED for database connection (preferred)
# Use a single connection string if possible. Example:
# FALKORDB_URL=redis://localhost:6379/0

# Optional: enable debug/reload when running the app directly
# FASTAPI_DEBUG=False

# REQUIRED for full functionality (OAuth, only if you use login flows)
# GOOGLE_CLIENT_ID=your_google_client_id
# GOOGLE_CLIENT_SECRET=your_google_client_secret
# GITHUB_CLIENT_ID=your_github_client_id

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FalkorDB/QueryWeaver](https://github.com/FalkorDB/QueryWeaver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
