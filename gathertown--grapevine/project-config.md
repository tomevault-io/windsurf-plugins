---
trigger: always_on
description: This file provides guidance to AI agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI agents when working with code in this repository.

## Project Overview

Grapevine is a real-time unified knowledge store that ingests information from various corporate knowledge sources (GitHub, Slack, Notion, Linear, Google Drive, and HubSpot) and provides semantic search capabilities through an MCP (Model Context Protocol) server. It's a collection of services working together (ingest-gatekeeper, MCP server, admin UI/API, Slack bot, ingest workers, index workers) to provide an MCP and Slack bot experience that understands all of the information at your company and helps you easily answer internal questions.

## Development Commands

### Environment Setup

```bash
# Install dependencies
mise install

# Start dev environment
mise dev
```

### Testing (Python)

Grapevine uses pytest for Python testing. Tests are provided via the test dependency group and should be run with uv:

```bash
# Install test dependencies
uv sync --group test

# Run tests
uv run pytest tests/ -v
```

Tests include MCP tool tests (keyword and semantic search) and use real backends (OpenSearch + Postgres), so ensure your environment is configured before running.

### Code Quality

```bash
# Install test dependencies (includes mypy and ruff)
uv sync --group test

# Run type checking
uv run mypy

# Run linting
uv run ruff check

# Run formatting
uv run ruff format

# Auto-fix linting issues
uv run ruff check --fix

# Auto-fix with unsafe fixes (for more aggressive fixes)
uv run ruff check --fix --unsafe-fixes
```

### Dead Code Detection

```bash
# Primary tool: Vulture (detects both globally unused and unreachable code)
uv run vulture

# Detect unreachable code with vulture (60%+ confidence)
uv run vulture --min-confidence 60

# Detect only 100% certain dead code with vulture
uv run vulture --min-confidence 100

# Run custom autofix script for 100% confident vulture issues
 scripts/vulture_autofix.py
```

**IMPORTANT**: Always run typing, linting, formatting, and dead code checks after making any code changes:

1. **Auto-fix lint issues first**: `uv run ruff check --fix` (try `--unsafe-fixes` if needed)
2. **Format code**: `uv run ruff format`
3. **Verify lint passes**: `uv run ruff check`
4. **Check types**: `uv run mypy`
5. **Check for dead code**: `uv run vulture`

These commands must be run after every code modification to maintain code quality standards.

### JavaScript/TypeScript Services Code Quality

For JavaScript/TypeScript services in `js-services/`:

```bash
# Navigate to the js-services directory
cd js-services

# Install dependencies
yarn install

# Run linting for all services
yarn lint

# Run linting for specific service
yarn nx run slack-bot:lint

# Auto-fix linting issues
yarn lint:fix

# Run type checking for all services
yarn type-check

# Format code for all services
yarn format

# Check formatting
yarn format:check

# Build all services
yarn build

# Build specific service
yarn nx run slack-bot:build

# Run development server with hot-reloading
yarn nx run slack-bot:serve:dev
```

**IMPORTANT**: Always run the following after making any code changes to JavaScript/TypeScript services:

1. **Format code**: `yarn format`
2. **Verify lint passes**: `yarn lint`
3. **Check types**: `yarn type-check`
4. **Build**: `yarn build`

These commands must be run after every code modification to maintain code quality standards.

### JavaScript/TypeScript Testing

For JavaScript/TypeScript services in `js-services/`, Jest is configured for unit testing:

```bash
# Navigate to the js-services directory
cd js-services

# Run all tests for all services
yarn test

# Run tests for specific service
yarn nx run admin-frontend:test
yarn nx run admin-backend:test
yarn nx run slack-bot:test

# Run tests in watch mode for development
yarn nx run admin-frontend:test:watch
yarn nx run admin-backend:test:watch

# Run tests with coverage reporting
yarn nx run admin-frontend:test:coverage
yarn nx run admin-backend:test:coverage
```

**Test Configuration**:

- Jest is configured via `jest.config.cjs` in each service directory
- Frontend tests use jsdom environment for React component testing
- Backend tests use Node.js environment for API and utility function testing
- Test files should be placed in `__tests__` directories alongside source files
- TypeScript support is configured automatically

**Test Coverage**:

- Domain validation functions have comprehensive test suites covering all edge cases
- Both frontend and backend validation logic are tested for consistency
- Tests include protocol stripping, subdomain handling, path removal, and error cases

### Running Services

#### Development

To start dev environment:

```bash
mise dev
```

## Database Migrations

The project uses a comprehensive migration system for both control and tenant databases:

```bash
# Create a new migration
mise migrations create tenant "add user preferences table"

# Run migrations locally
mise migrations migrate --control --all-tenants

# Check migration status
mise migrations status

# List existing migrations
mise run migrations list

# Mark/unmark migrations (for manual maintenance)
mise migrations mark --apply 20250828000000 --control

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gathertown/grapevine](https://github.com/gathertown/grapevine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
