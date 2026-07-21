---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Project Overview

Langflow is a visual workflow builder for AI-powered agents. It has a Python/FastAPI backend, React/TypeScript frontend, and a lightweight executor CLI (lfx).

## Prerequisites

- **Python:** 3.10-3.14
- **uv:** >=0.4 (Python package manager)
- **Node.js:** >=20.19.0 (v22.12 LTS recommended)
- **npm:** v10.9+
- **make:** For build coordination

## Common Commands

### Development Setup
```bash
make init              # Install all dependencies + pre-commit hooks
make run_cli           # Build and run Langflow (http://localhost:7860)
make run_clic          # Clean build and run (use when frontend issues occur)
```

### Development Mode (Hot Reload)
```bash
make backend           # FastAPI on port 7860 (terminal 1)
make frontend          # Vite dev server on port 3000 (terminal 2)
```

For component development, enable dynamic loading:
```bash
LFX_DEV=1 make backend                    # Load all components dynamically
LFX_DEV=mistral,openai make backend       # Load only specific modules
```

### Code Quality
```bash
make format_backend    # Format Python (ruff) - run FIRST before lint
make format_frontend   # Format TypeScript (biome)
make format            # Both
make lint              # mypy type checking
```

### Testing
```bash
make unit_tests                    # Backend unit tests (pytest, parallel)
make unit_tests async=false        # Sequential tests
uv run pytest path/to/test.py      # Single test file
uv run pytest path/to/test.py::test_name  # Single test

make test_frontend                 # Jest unit tests
make tests_frontend                # Playwright e2e tests
```

### Database Migrations
```bash
make alembic-revision message="Description"  # Create migration
make alembic-upgrade                         # Apply migrations
make alembic-downgrade                       # Rollback one version
```

## Architecture

### Monorepo Structure
```
src/
├── backend/
│   ├── base/langflow/     # Core backend package (langflow-base)
│   │   ├── api/           # FastAPI routes (v1/, v2/)
│   │   ├── components/    # Built-in Langflow components
│   │   ├── services/      # Service layer (auth, database, cache, etc.)
│   │   ├── graph/         # Flow graph execution engine
│   │   └── custom/        # Custom component framework
│   └── tests/             # Backend tests
├── frontend/              # React/TypeScript UI
│   └── src/
│       ├── components/    # UI components
│       ├── stores/        # Zustand state management
│       └── icons/         # Component icons
└── lfx/                   # Lightweight executor CLI
```

### Key Packages
- **langflow**: Main package with all integrations
- **langflow-base**: Core framework (api, services, graph engine)
- **lfx**: Standalone CLI for running flows (`lfx serve`, `lfx run`)

### Service Layer
Backend services in `src/backend/base/langflow/services/`:
- `auth/` - Authentication
- `authorization/` - Authorization (RBAC) plugin layer — see below
- `database/` - SQLAlchemy models and migrations
- `cache/` - Caching layer
- `storage/` - File storage
- `tracing/` - Observability integrations

### Authorization (RBAC)

Authorization is a pluggable layer separate from authentication:

- **OSS** ships the interface (`BaseAuthorizationService` in `lfx`) + a pass-through implementation (`LangflowAuthorizationService`) + the `authz_*` and `casbin_rule` DB schema + route guards.
- Implementations register via the `lfx.services` entry point `authorization_service` in `lfx.toml` (same pattern as the SSO `auth_service`). A registered plugin reads the `authz_*` admin tables and writes compiled rules to `casbin_rule`.

Default is **off**: `LANGFLOW_AUTHZ_ENABLED=false`. When enabled with only the OSS stub registered, every check returns allow — the stub is a no-op so routes stay wired and audit rows still flow. Real allow/deny requires a registered authorization plugin.

Route guards live in `langflow.services.authorization.guards` (the legacy `langflow.services.authorization.utils` path re-exports them for backward compatibility):
- `ensure_flow_permission(user, FlowAction.*, flow_id=..., flow_user_id=..., workspace_id=..., folder_id=...)` — single-flow CRUD + execute
- `ensure_deployment_permission(user, DeploymentAction.*, deployment_id=..., deployment_user_id=..., workspace_id=..., project_id=...)`
- `ensure_project_permission(user, ProjectAction.*, project_id=..., project_user_id=..., workspace_id=...)`
- `ensure_knowledge_base_permission(user, KnowledgeBaseAction.*, kb_name=..., kb_user_id=...)`
- `ensure_variable_permission(user, VariableAction.*, variable_id=..., variable_user_id=...)`
- `ensure_file_permission(user, FileAction.*, file_id=..., file_user_id=...)`
- `ensure_share_permission(user, ShareAction.*, share_id=..., share_user_id=...)`
- `filter_visible_resources(user, resource_type=..., candidates=..., act=...)` — list-endpoint filter; safe no-op in OSS

The enforcement request shape is `(subject, domain, object, action)`:
- subject = `user:{uuid}`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [langflow-ai/langflow](https://github.com/langflow-ai/langflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
