---
trigger: always_on
description: Always follow these instructions and only fallback to additional search and context gathering if the information in these instructions is incomplete or found to be in error.
---

# Boxtribute Copilot Instructions

Always follow these instructions and only fallback to additional search and context gathering if the information in these instructions is incomplete or found to be in error.

## Project Overview

Boxtribute is a humanitarian relief web application supporting the distribution of over 1 million items annually across 15+ locations in Europe and the Middle East. The application consists of:

- **Frontend**: React TypeScript app built with Vite, Chakra UI, Apollo Client
- **Backend**: Python Flask GraphQL API using Ariadne, Peewee ORM, MySQL
- **Additional Apps**: Statistics visualization (shared-front), shared components
- **Architecture**: Docker-based development, deployed on Google App Engine

### Architecture Diagrams
For detailed system architecture understanding:
- **System Landscape**: `docs/c4-system-landscape.png` - High-level overview of system components and external integrations
- **Backend Components**: `docs/c4-backend-components.jpg` - Internal backend service architecture and data flow

## General Instructions

**Think first and make a plan before you start implementing.** Always analyze the problem, understand the codebase, and create a clear implementation plan before making any changes.

**CRITICAL**: NEVER modify `.circleci/config.yml` or trigger any deployment manually. If changes to CI configuration are absolutely necessary, request approval in a PR comment.

**Always report issues with building/testing in the PR.** If you encounter build failures, test failures, or other issues during development, document them clearly in your progress reports so stakeholders are aware of any blockers or limitations.

## Essential Setup Commands

**Note**: These instructions complement the automated setup defined in `.github/workflows/copilot-setup-steps.yml`. The workflow file handles dependency installation in the Copilot environment, while these instructions are for manual development setup.

Run these commands exactly in order for initial setup:

```bash
# Clone and enter repository
cd /path/to/boxtribute

# Copy environment configuration (REQUIRED)
cp example.env .env

# Install pnpm globally (Node.js package manager)
npm install -g pnpm@9.15.1

# Install all dependencies for the workspace
pnpm install

# Create Python virtual environment for backend development
python3 -m venv .venv
source .venv/bin/activate

# Install backend dependencies (requires internet access to PyPI)
pip install -U -e back -r back/requirements-dev.txt
```

## Development Environment

### Docker Compose (Primary Method)

Start the complete development environment:

```bash
# Start all services (database, backend, frontend, shared-front)
docker compose up

# NEVER CANCEL: Initial Docker build can take 15-30 minutes
# Set timeout to 45+ minutes for first-time builds
# Subsequent starts are much faster (1-2 minutes)

# Access points after startup:
# - Frontend: http://localhost:3000
# - Backend GraphQL: http://localhost:5005/graphql
# - Database: localhost:32000 (MySQL, user: root, password: dropapp_root)
```

**CRITICAL**: If you encounter network timeouts with npmjs.org or pypi.org during Docker builds, this is a known environment limitation. Document the specific error and proceed with local development approach.

### Local Development (Alternative)

If Docker Compose fails due to network issues:

```bash
# Start database only
docker compose up -d db

# Frontend development server (in separate terminal)
cd front
pnpm dev
# Runs on http://localhost:3000

# Backend development server (in separate terminal)
cd back
source ../.venv/bin/activate
MYSQL_HOST=127.0.0.1 MYSQL_USER=root MYSQL_PASSWORD=dropapp_root MYSQL_DB=dropapp_dev MYSQL_PORT=32000 python -m boxtribute_server.dev_main
# Runs on http://localhost:5005
```

## Testing Commands

### Frontend Tests
```bash
# Run all frontend tests with coverage
# NEVER CANCEL: Takes ~1-2 minutes to complete
pnpm test:coverage
# Timeout: Set to 5+ minutes

# Run tests in watch mode for development
pnpm test

# Test shared components separately
pnpm -C shared-components test:coverage
```

### Backend Tests
```bash
# Start database for testing
docker compose up -d db

# Install backend dependencies (REQUIRES internet access to PyPI)
cd back
source ../.venv/bin/activate
pip install -U -e . -r requirements-dev.txt
git config --unset-all core.hooksPath
pre-commit install --overwrite
# NOTE: May fail with network timeouts in restricted environments

# Run backend tests (requires MySQL database and dependencies)
pytest --ignore=test/auth0_integration_tests/
# NEVER CANCEL: Takes ~5-10 minutes including database setup
# Timeout: Set to 15+ minutes

# Run auth0_integration_tests separately if needed (requires repository secrets)
pytest test/auth0_integration_tests/
# NOTE: Requires TEST_AUTH0_MANAGEMENT_API_CLIENT_SECRET and TEST_AUTH0_CLIENT_SECRET from repository secrets

# Run specific test categories (when dependencies are available)
pytest test/unit_tests/          # Unit tests only (~2 minutes)
pytest test/model_tests/         # Data model tests (~3 minutes)  
pytest test/endpoint_tests/      # API endpoint tests (~8 minutes)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [boxwise/boxtribute](https://github.com/boxwise/boxtribute) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
