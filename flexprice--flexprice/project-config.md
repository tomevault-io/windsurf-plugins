---
trigger: always_on
description: This file provides guidance to WARP (warp.dev) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Quick Start Commands

### Development Setup

```bash
# Complete development environment setup (Docker-based)
make dev-setup

# Run application locally (requires infrastructure running)
go run cmd/server/main.go

# Start only infrastructure services
docker compose up -d postgres kafka clickhouse temporal temporal-ui
```

### Running the Application

The application supports multiple deployment modes via `FLEXPRICE_DEPLOYMENT_MODE` environment variable:

- `local` - Runs all services (API, Consumer, Worker) in a single process
- `api` - Runs only the API server
- `consumer` - Runs only the Kafka consumer for event processing
- `temporal_worker` - Runs only Temporal workflow workers

```bash
# Run in local mode (default)
make run-server

# Using Docker Compose
make up  # Start all services
make down  # Stop all services
make restart-flexprice  # Restart only FlexPrice services (not infrastructure)
```

### Testing

```bash
# Run all tests
make test

# Run tests with coverage
make test-coverage

# Run tests verbosely
make test-verbose
```

### Database Operations

```bash
# Generate Ent code from schema
make generate-ent

# Run database migrations
make migrate-ent

# Dry-run migrations (see SQL without executing)
make migrate-ent-dry-run

# Generate migration file
make generate-migration

# Run PostgreSQL migrations only
make migrate-postgres

# Run ClickHouse migrations only
make migrate-clickhouse
```

### API Documentation

```bash
# Generate Swagger documentation
make swagger

# Generates both Swagger 2.0 and OpenAPI 3.0 specs in docs/swagger/
```

### SDK Generation

SDKs and the MCP server are generated from the OpenAPI spec. Output layout: **api/** (api/go, api/typescript, api/python, api/mcp).

**Source:** [docs/swagger/swagger-3-0.json](docs/swagger/swagger-3-0.json) (regenerate with `make swagger`).

**Commands:**

```bash
# Single command: validate + generate all SDKs/MCP + merge custom (uses existing docs/swagger/swagger-3-0.json)
make sdk-all

# When you change the API, regenerate the spec first, then run sdk-all
make swagger
make sdk-all

# Validate OpenAPI
make speakeasy-validate

# Generate Go SDK (validate + generate + custom merge + build; uses existing swagger)
make go-sdk

# Quick regeneration (no clean)
make regenerate-go-sdk

# Generate all targets (after configuring workflow targets)
make swagger speakeasy-generate
make merge-custom

# Merge custom files only (after any SDK generation run)
make merge-custom
```

**Custom methods and files:** Custom logic lives in `api/custom/<lang>/` (same path structure as api/<lang>/). It is merged into the generated output after every generation via `make merge-custom`. Do not edit generated files under api/<lang>/ for custom code; edit the custom tree so changes survive regeneration. See [api/custom/README.md](api/custom/README.md). READMEs for each SDK and MCP are maintained in `api/custom/<lang>/README.md` and overwrite the generated README on merge; `api/go`, `api/python`, and `api/typescript` also list README in `.genignore` so a generate run without merge-custom does not overwrite the current README.

**MCP server:** Generated in **api/mcp**. Run from that directory (e.g. `npx . start` or per generated README). Auth: set `FLEXPRICE_API_KEY` or the env var documented in the MCP server README. For large tool sets, use dynamic mode (e.g. `--mode dynamic`) to reduce context size; document in api/mcp README. Only operations whose OpenAPI tags are listed in the MCP allowed-tags configuration are included; the filtered spec is built by `make filter-mcp-spec` (runs automatically before `make sdk-all`). To change which tools are exposed, edit `.speakeasy/mcp/allowed-tags.yaml` and run `make filter-mcp-spec` then `make sdk-all`.

**SDK integration tests:** In **api/tests/** – tests for published SDKs only. Repos: Go [go-sdk](https://github.com/flexprice/go-sdk), Python [python-sdk](https://github.com/flexprice/python-sdk), TypeScript [javascript-sdk](https://github.com/flexprice/javascript-sdk), MCP [mcp-server](https://github.com/flexprice/mcp-server). Published packages: `pip install flexprice`, `npm i @flexprice/sdk`, `npm i @flexprice/mcp-server`. Run `make test-sdk` or `make test-sdks`; see [api/tests/README.md](api/tests/README.md). Context: [SDK PR #1288](https://github.com/flexprice/flexprice/pull/1288).

**Publishing:** Single workflow [.github/workflows/generate-sdks.yml](.github/workflows/generate-sdks.yml): on push to main (path-filtered) or workflow_dispatch it runs generate → push to GitHub repos → publish to npm/PyPI. Secrets: `SPEAKEASY_API_KEY`, `SDK_DEPLOY_GIT_TOKEN`, `NPM_TOKEN`, `PYPI_TOKEN`. See [api/README.md](api/README.md#publishing). To test the full pipeline (including artifact upload), run on GitHub; local `act` runs often fail at upload-artifact due to missing `ACTIONS_RUNTIME_TOKEN`.

**Best practices checklist (per release):**

| Area           | Practices                                                                                                              |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flexprice/flexprice](https://github.com/flexprice/flexprice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
