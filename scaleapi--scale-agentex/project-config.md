---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

Agentex is a comprehensive platform for building and deploying intelligent agents. This repository contains:

- **agentex/** - Backend services (FastAPI + Temporal workflows)
- **agentex-ui/** - Developer UI for interacting with agents

The platform integrates with the separate [agentex-python SDK](https://github.com/scaleapi/scale-agentex-python) for creating and running agents.

## Development Environment Setup

### Prerequisites

- Python 3.12+ (required for agentex-sdk)
- Docker and Docker Compose
- Node.js (for frontend)
- uv (Python package manager)

### Quick Start (Recommended)

One command does everything (auto-installs prerequisites if missing):

```bash
./dev.sh                    # Installs deps + starts backend + frontend
```

> Make sure Docker Desktop or Rancher Desktop is running first.

Other commands:
```bash
./dev.sh stop               # Stop all services
./dev.sh status             # Check service status
./dev.sh logs               # View all logs
./dev.sh restart            # Restart all services
```

**Then in a separate terminal - Agent Development:**
```bash
agentex init                # Create a new agent
cd your-agent-name/
uv venv && source .venv/bin/activate && uv sync
agentex agents run --manifest manifest.yaml
```

### Manual Setup (Alternative - 3 Terminals)

**Terminal 1 - Backend:**
```bash
cd agentex/
make dev                    # Starts Docker services and backend
```

**Terminal 2 - Frontend:**
```bash
cd agentex-ui/
npm install
npm run dev                 # Starts Next.js dev server
```

**Terminal 3 - Agent Development:**
```bash
agentex init                # Create a new agent
cd your-agent-name/
uv venv && source .venv/bin/activate && uv sync
agentex agents run --manifest manifest.yaml
```

### Backend Services (Docker Compose)

When running `make dev` in agentex/, the following services start:

- **Port 5003**: FastAPI backend server
- **Port 5432**: PostgreSQL (application database)
- **Port 5433**: PostgreSQL (Temporal database)
- **Port 6379**: Redis (streams and caching)
- **Port 27017**: MongoDB (document storage)
- **Port 7233**: Temporal server
- **Port 8080**: Temporal Web UI

All services are networked via `agentex-network` bridge network.

## Common Development Commands

### Backend (agentex/)

```bash
# Setup and installation
make install              # Install dependencies with uv
make install-dev          # Install with dev dependencies (includes pre-commit)
make clean                # Clean venv and lock files

# Development server
make dev                  # Start all Docker services
make dev-stop             # Stop Docker services
make dev-wipe             # Stop services and wipe volumes

# Database migrations
make migration NAME="description"  # Create new Alembic migration
make apply-migrations              # Apply pending migrations

# Testing
make test                                    # Run all tests
make test FILE=tests/unit/                   # Run unit tests
make test FILE=tests/unit/test_foo.py        # Run specific test file
make test NAME=crud                          # Run tests matching pattern
make test-unit                               # Unit tests shortcut
make test-integration                        # Integration tests shortcut
make test-cov                                # Run with coverage report
make test-docker-check                       # Verify Docker setup for tests

# Linting (ruff)
uv run ruff check src/                       # Check for lint errors
uv run ruff check src/ --fix                 # Auto-fix lint errors
uv run ruff format src/                      # Format code
uv run ruff check path/to/file.py            # Check specific file

# Documentation
make serve-docs           # Serve MkDocs on localhost:8001
make build-docs           # Build documentation

# Deployment
make docker-build         # Build production Docker image
```

### Frontend (agentex-ui/)

```bash
npm install               # Install npm dependencies
npm run dev               # Next.js dev server with Turbopack
npm run build             # Build production bundle
npm run typecheck         # TypeScript type checking
npm run lint              # Run ESLint
npm run format            # Run Prettier formatting
npm test                  # Run tests
```

### Agent Development (agentex-sdk)

```bash
# Always set this first
export ENVIRONMENT=development

# Agent management
agentex init                                      # Create new agent
agentex agents run --manifest manifest.yaml       # Run agent locally (dev)
agentex agents list                               # List all agents
agentex agents build --manifest manifest.yaml --push   # Build & push image
agentex agents deploy --manifest manifest.yaml         # Deploy to staging

# Package management (if using uv in agent)
agentex uv sync           # Sync dependencies
agentex uv add requests   # Add new dependency

# Other utilities
agentex tasks list        # View agent tasks
agentex secrets create    # Manage secrets
```

## Architecture

### Domain-Driven Design Structure


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scaleapi/scale-agentex](https://github.com/scaleapi/scale-agentex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
