---
trigger: always_on
description: **qontract-reconcile** is a comprehensive infrastructure reconciliation tool developed by Red Hat's App-SRE Team. It manages 80+ integrations across AWS, OpenShift, GitHub, GitLab, and other platforms, following a declarative "desired state vs current state" reconciliation pattern.
---

# AGENTS.md

## Project Overview

**qontract-reconcile** is a comprehensive infrastructure reconciliation tool developed by Red Hat's App-SRE Team. It manages 80+ integrations across AWS, OpenShift, GitHub, GitLab, and other platforms, following a declarative "desired state vs current state" reconciliation pattern.

## Workspace Structure

Four packages in a uv workspace with strict boundaries:

| Package                | Role                             | Imports from                            |
| ---------------------- | -------------------------------- | --------------------------------------- |
| `reconcile/`           | 158+ CLI integrations (original) | `qontract_utils`, `qontract_api_client` |
| `qontract_api/`        | FastAPI REST service             | `qontract_utils`, `qontract_api_client` |
| `qontract_api_client/` | Auto-generated API client        | nothing (standalone)                    |
| `qontract_utils/`      | Shared utilities library         | nothing (standalone)                    |

### Import Rules (ADR-007)

- `qontract_api/` MUST NOT import from `reconcile/`
- `reconcile/` MUST NOT import from `qontract_api/`
- Shared code goes to `qontract_utils/` — migrated from `reconcile/utils/` with refactoring, type hints, tests (>80% coverage)
- `reconcile/` is a read-only reference implementation for qontract-api development

### Generated Files — NEVER Edit Manually

1. **`qontract_api/openapi.json`** — generated from FastAPI code via `cd qontract_api && make generate-openapi-spec`
2. **`qontract_api_client/qontract_api_client/`** — generated from openapi.json via `cd qontract_api_client && make generate-client`
3. **`reconcile/gql_definitions/**/*.py`** — generated from `.gql` files via `make gql-query-classes`

Change the SOURCE, then regenerate. Never edit these files.

### Key Directories

- `reconcile/` — Original CLI integrations (read-only reference for qontract-api work)
- `reconcile/gql_definitions/` — Auto-generated GraphQL dataclasses
- `qontract_api/qontract_api/integrations/` — API-based integration services (Layer 3)
- `qontract_api/qontract_api/routers/` — FastAPI route definitions
- `qontract_api/qontract_api/tasks/` — Celery task definitions
- `qontract_api/qontract_api/cache/` — Distributed cache backends
- `qontract_utils/qontract_utils/vcs/` — VCS clients (GitHub, GitLab — Layer 1)
- `qontract_utils/qontract_utils/secret_reader/` — Vault secret readers
- `tools/` — CLI utilities and standalone tools
- `docs/adr/` — Architecture Decision Records (binding, read before architectural changes)
- `docs/patterns/` — Implementation pattern documentation
- `helm/qontract-reconcile/` — Helm chart for deployment

## Development Environment

### Prerequisites

- **Python 3.12** (exact pin)
- **uv** (>=0.10.11) for dependency management
- **Docker/Podman** for containerized development

### Setup

```bash
uv sync -U               # Install dependencies
```

## Essential Commands

### Code Quality

```bash
make format              # Auto-fix: ruff check --fix + ruff format
make linter-test         # Lint: ruff check --no-fix + ruff format --check
make types-test          # Type check: mypy strict (disallow_untyped_defs, pydantic plugin)
make unittest            # Tests: pytest with coverage (>60% required)
make all-tests           # ALL: linter-test + types-test + qenerate-test + helm-test + unittest
```

Sub-packages have their own `make test` (in `qontract_api/`, `qontract_api_client/`, `qontract_utils/`).

### Regeneration Workflows

After changing FastAPI routes/models:

```bash
cd qontract_api && make generate-openapi-spec   # → updates openapi.json
cd qontract_api_client && make generate-client   # → regenerates client from openapi.json
```

After changing GraphQL schema:

```bash
make gql-introspection     # Fetch schema from localhost:4000/graphql
make gql-query-classes     # Generate Pydantic models from .gql files
# or combined:
make qenerate
```

### Development Workflows

```bash
make dev-reconcile-loop  # Start containerized development environment
```

## Tooling Stack

- **Package manager**: uv, workspace with 4 members
- **Linting/formatting**: ruff (line-length=88, target py312)
- **Type checking**: mypy strict (disallow_untyped_defs, pydantic plugin)
- **Testing**: pytest + pytest-mock + moto (AWS mocking) + responses (HTTP mocking)
- **GraphQL codegen**: qenerate
- **API client codegen**: openapi-python-client with custom templates
- **Build backend**: hatchling + uv-dynamic-versioning (git tags)
- **CI/CD**: Tekton pipelines (.tekton/), separate pipelines per package
- **Container**: podman/docker, multi-stage Dockerfile

### Ruff Config Differences

- **Root** (`reconcile/`, `tools/`): Selected rule sets (E, W, F, I, PL, UP, SIM, B, PERF, etc.)
- **qontract_api/** and **qontract_utils/**: `select = ["ALL"]` with pragmatic ignores
- All exclude `reconcile/gql_definitions` from linting

### Key Entry Points

- `qontract-reconcile` → `reconcile.cli:integration`
- `qontract-cli` → `tools.qontract_cli:root`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [app-sre/qontract-reconcile](https://github.com/app-sre/qontract-reconcile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
