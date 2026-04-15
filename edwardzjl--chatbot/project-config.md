---
trigger: always_on
description: **Chatbot** is a Single-page application (SPA) that recreates key features of popular LLM-based chatbots like ChatGPT, Gemini, and Le Chat. It consists of a **FastAPI backend** (Python) and **React frontend** (JavaScript/Vite) with support for tool usage, multimodal inputs, reasoning models, and working memory.
---

# Copilot Instructions for Chatbot Repository

## Repository Overview

**Chatbot** is a Single-page application (SPA) that recreates key features of popular LLM-based chatbots like ChatGPT, Gemini, and Le Chat. It consists of a **FastAPI backend** (Python) and **React frontend** (JavaScript/Vite) with support for tool usage, multimodal inputs, reasoning models, and working memory.

### High-Level Architecture
- **Backend**: FastAPI with LangChain, SQLAlchemy, and LangGraph for agent workflows
- **Frontend**: React 19 with Material-UI, Vite build system, and WebSocket support
- **Database**: SQLite (development) / PostgreSQL (production) with Alembic migrations
- **Deployment**: Docker containers with Kubernetes support (vanilla and Knative)
- **Authentication**: OpenID Connect via external OAuth clients

### Project Structure
```
/api/           - FastAPI backend with Python 3.13
/web/           - React frontend with Node.js LTS
/manifests/     - Kubernetes deployment configurations  
/.github/       - CI/CD workflows and issue templates
/.devcontainer/ - VS Code development container setup
```

## Build Requirements & Setup

### Prerequisites
- **Python 3.13** (3.12 works with warnings)
- **Node.js LTS** (v22+)
- **uv** for Python dependency management
- **Yarn 4.9.2** via corepack for Node.js dependencies

### Initial Setup Commands
```bash
# API setup
cd api
pip install uv
uv sync  # Install development dependencies

# Web setup  
cd web
corepack enable
yarn install --immutable
```

## Build, Test, and Lint Commands

### API (Python/FastAPI)
```bash
cd api

# Install dependencies
uv sync

# Linting (uses ruff)
make lint
# OR: uv run ruff check && uv run ruff format --check

# Formatting
make format  
# OR: uv run ruff check --fix && uv run ruff format

# Testing
make test
# OR: uv run python -m unittest

# Database migrations (requires PostgreSQL connection)
uv run alembic check
uv run alembic upgrade head
```

### Web (React/Vite)
```bash
cd web

# Install dependencies
corepack enable
yarn install --immutable

# Linting (uses ESLint)
make lint
# OR: yarn lint

# Formatting
make format
# OR: yarn format  

# Testing
make test
# OR: yarn test --run

# Building for production
make build
# OR: yarn build

# Development server
yarn dev  # Runs on port 3000 with API proxy
```

## Known Issues & Workarounds


### Web Build Warnings
- **Issue**: Vite warns about chunks >500KB after minification
- **Status**: Known issue, not breaking - application builds and runs correctly
- **Details**: Main bundle is ~1.87MB, likely due to Material-UI and React dependencies


### Database Requirements
- **Development**: SQLite (default, no setup needed)
- **Production**: PostgreSQL required for full feature set
- **CI Limitation**: Database check job disabled due to external dependency requirements

## Continuous Integration

The repository uses GitHub Actions with separate workflows:

### API CI (`.github/workflows/api-ci.yml`)
- **Triggers**: Changes to `api/**` or workflow file
- **Jobs**: lint, test, check-db (disabled)
- **Runtime**: Ubuntu latest with Python 3.13
- **Dependencies**: `uv sync`

### Web CI (`.github/workflows/web-ci.yml`)  
- **Triggers**: Changes to `web/**` or workflow file
- **Jobs**: lint, test, build
- **Runtime**: Ubuntu latest with Node.js LTS
- **Dependencies**: `corepack enable && yarn install --immutable`

### Pre-commit Hooks
Configuration in `.pre-commit-config.yaml`:
- **check-yaml, end-of-file-fixer, trailing-whitespace** from pre-commit-hooks
- **ruff linting and formatting** for Python files in `api/`
- **Note**: Pre-commit must be installed separately: `pip install pre-commit && pre-commit install`

## Configuration Files

### Python/API Configuration
- **`api/pyproject.toml`**: Dependencies and project configuration
- **`api/uv.lock`**: Locked dependency versions for reproducible builds
- **`api/ruff.toml`**: Minimal ruff configuration for flake8-type-checking
- **`api/alembic.ini`**: Database migration configuration with PostgreSQL default
- **`api/Makefile`**: Build commands (lint, format, test)

### JavaScript/Web Configuration  
- **`web/package.json`**: Dependencies and npm scripts
- **`web/eslint.config.js`**: ESLint v9 flat config with React plugins
- **`web/vite.config.js`**: Build config with proxy setup for API during development
- **`web/vitest.config.js`**: Test configuration
- **`web/Makefile`**: Build commands (lint, format, test, build)

## Development Environment

### Local Development
```bash
# Start API server (port 8000)
cd api
uv run uvicorn chatbot.main:app --reload

# Start web dev server (port 3000)  
cd web
yarn dev  # Includes API proxy configuration
```

### Docker Development
- **Dockerfile**: Multi-stage build (frontend builder → backend builder → final app)
- **Dev Container**: VS Code devcontainer with Python 3.13 base image
- **Compose**: Simple service definition in `.devcontainer/docker-compose.yml`

### Environment Variables
Key configuration via environment variables (see README for full list):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/edwardzjl) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
