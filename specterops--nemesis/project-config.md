---
trigger: always_on
description: **NEVER read, open, or access `.env` files under any circumstances.** This includes:
---

# Repository Guidelines

## File Restrictions

**NEVER read, open, or access `.env` files under any circumstances.** This includes:
- `.env`
- `.env.local`
- `.env.development`
- `.env.production`
- Any file matching the pattern `.env*`

If you need environment variable information, refer to `env.example` instead.

## Project Overview

Nemesis is an open-source, centralized data processing platform (v2.0) that ingests, enriches, and enables collaborative analysis of files collected during offensive security assessments. Built on Docker with Dapr integration, it functions as an "offensive VirusTotal."

## Project Structure & Module Organization

Nemesis is organized as a multi-service monorepo:
- `projects/`: deployable services (for example `web_api`, `file_enrichment`, `frontend`, `agents`), each typically with its own `pyproject.toml` and `tests/`.
- `libs/`: shared Python libraries (`common`, `file_linking`, `nemesis_dpapi`, `file_enrichment_modules`).
- `infra/`: Docker, Dapr, Traefik, and observability configuration.
- `tools/`: repo-level developer scripts (`install_dev_env.sh`, `lint.sh`, `test.sh`, `nemesis-ctl.sh`).
- `docs/`: MkDocs content and generated API documentation assets.

## Common Commands

### Running Nemesis

```bash
# Start production services
./tools/nemesis-ctl.sh start prod

# Start with monitoring, jupyter, and LLM support
./tools/nemesis-ctl.sh start prod --monitoring --jupyter --llm

# Start development environment (always builds)
./tools/nemesis-ctl.sh start dev

# Stop services (use same flags as start)
./tools/nemesis-ctl.sh stop prod --monitoring --jupyter --llm

# Stop and remove volumes
./tools/nemesis-ctl.sh clean prod --monitoring --jupyter --llm
```

### Development Setup

```bash
# Install all uv dependencies across projects
./tools/install_dev_env.sh

# Install dependencies for a single project
cd projects/web_api && uv sync
```

### Package Management

For adding, upgrading, or removing packages, use the `$managing-packages` skill.

### Linting & Formatting (Ruff)

Run linting/formatting after each fix/change.
```bash
# Check all Python code (configured in root pyproject.toml)
uv run ruff check . --fix

# Format code
uv run ruff format .

# Check specific project
cd projects/web_api && uv run ruff check . --fix
```

### Testing

```bash
# Run tests for a specific project
cd projects/file_enrichment && uv run pytest tests/

# Run single test file
cd projects/web_api && uv run pytest tests/test_specific.py

# Run specific test
cd projects/web_api && uv run pytest tests/test_file.py::test_function_name
```

### Docker Commands

Information about building dev/prod container images can be found in the [docker compose docs](./docs/docker_compose.md).

General instructions for dev docker containers/images:
```bash
# Build base images first (required before building services)
docker compose -f compose.base.yaml build

# View logs for a service
docker compose logs -f web-api

# Rebuild Dapr-enabled services (must include the -dapr sidecar)
docker compose up -d --build file-enrichment file-enrichment-dapr
docker compose up -d --build web-api web-api-dapr
```

**Note:** Services using Dapr (file-enrichment, web-api, alerting, etc.) have a companion `-dapr` sidecar container. When rebuilding these services, always restart both the service and its Dapr sidecar to ensure proper communication.

## Testing Guidelines

- **Write tests for every new feature.** New functionality must include tests before it is considered complete.
- **Cover both happy path and unhappy path cases.** Tests should verify correct behavior with valid inputs (positive cases) and proper error handling with invalid inputs, missing data, edge cases, and failure conditions (negative cases).
- **Mock external services.** Do not make real calls to external dependencies (Dapr, SeaweedFS, PostgreSQL, RabbitMQ, etc.) in unit tests. Use mocks or fakes to isolate the code under test.

## Architecture

### Tech Stack
- **Python 3.13** with uv for dependency management
- **FastAPI** for REST services
- **React 18 + Vite + TypeScript** for frontend
- **PostgreSQL** for database
- **Dapr** for pub/sub, workflows, secrets, and service invocation
- **RabbitMQ** for message queue
- **SeaweedFS** for object storage
- **Traefik** for reverse proxy

### Directory Structure

```
projects/           # Microservices
├── web_api/        # FastAPI REST gateway (port 8000)
├── file_enrichment/# File analysis orchestration with Dapr workflows
├── document_conversion/ # Text extraction, PDF conversion
├── cli/            # Command-line file submission tool
├── frontend/       # React web UI
├── alerting/       # Apprise-based notification service
├── housekeeping/   # Cleanup/retention service
├── agents/         # AI-powered alert triage
├── jupyter/        # Jupyter notebook service
├── dotnet_service/ # .NET assembly analysis service
├── titus_scanner/  # Secret scanning with Titus
├── velociraptor_connector/ # Velociraptor integration
└── ...

libs/               # Shared Python libraries
├── common/         # DB connections, logging, models, Dapr wrappers
├── file_enrichment_modules/ # 20+ enrichment module implementations
├── chromium/       # Chrome/Edge data extraction

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SpecterOps/Nemesis](https://github.com/SpecterOps/Nemesis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
