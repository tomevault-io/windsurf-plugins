---
trigger: always_on
description: <!-- GSD:project-start source:PROJECT.md -->
---

<!-- GSD:project-start source:PROJECT.md -->
## Project

**MLflow OIDC Auth — Organization Support**

An MLflow authentication and authorization plugin (mlflow-oidc-auth) that adds OIDC-based login, RBAC with users/groups, and per-resource permission management to MLflow tracking servers. The project is adding multi-tenant organization support, aligning with MLflow 3.10's new organization features, to enable resource isolation across teams or external organizations sharing a single MLflow instance.

**Core Value:** Multi-tenant resource isolation — organizations must be able to share an MLflow instance while each tenant sees only their own experiments, models, and resources, with no accidental data leakage between tenants.

### Constraints

- **Compatibility**: Must target MLflow >=3.10.0 — org features require 3.10 baseline
- **Production impact**: Existing deployments must have a clear migration path, even if major version bump
- **Tech stack**: Python/FastAPI/Flask/SQLAlchemy backend, React/TypeScript frontend — no new frameworks
- **Plugin boundary**: Can only control auth/authz — cannot modify MLflow core behavior
- **Research dependency**: Implementation scope gated on research findings about MLflow 3.10 internals
<!-- GSD:project-end -->

<!-- GSD:stack-start source:codebase/STACK.md -->
## Technology Stack

## Languages
- Python 3.12 - Backend server, auth plugin, CLI, database models, API routers (`.python-version`, CI uses `python-version: 3.12`)
- TypeScript ~5.9 - React frontend UI (`web-react/package.json`)
- SQL - Database migrations via Alembic (`mlflow_oidc_auth/db/migrations/`)
- Bash - Dev/release scripts (`scripts/release.sh`, `scripts/run-dev-server.sh`)
- YAML - CI/CD workflows, Docker Compose (`scripts/docker-compose.yaml`, `.github/workflows/`)
## Runtime
- Python >=3.10 (declared in `pyproject.toml`), 3.12 used in development/CI
- Tox test matrix targets `py314` (`tox.ini` envlist)
- Node 24 (CI uses `node-version: 24` in `.github/workflows/unit-tests.yml`)
- ES modules (`"type": "module"` in `web-react/package.json`)
- pip + setuptools for Python (`pyproject.toml` build-system)
- Yarn for JavaScript (`web-react/yarn.lock` present)
- Lockfile: `web-react/yarn.lock` present; no Python lockfile (pip freeze / tox managed)
## Frameworks
- FastAPI >=0.132.0 - Primary ASGI application framework (`pyproject.toml`, `mlflow_oidc_auth/app.py`)
- Flask <4 - MLflow's built-in web framework, mounted as WSGI under FastAPI (`mlflow_oidc_auth/app.py`)
- Starlette - Underlying ASGI framework (via FastAPI), session middleware (`starlette.middleware.sessions`)
- MLflow >=3.10.0, <4 - ML experiment tracking server; this project is an auth plugin (`pyproject.toml`)
- React 19.1 - UI framework (`web-react/package.json`)
- React Router 7.9 - Client-side routing (`web-react/package.json`)
- Tailwind CSS 4 - Utility-first CSS framework (`@tailwindcss/vite` plugin in `web-react/package.json`)
- FontAwesome 7 - Icon library (`@fortawesome/*` packages)
- pytest >=8.3.2 - Python test runner (`pyproject.toml`)
- pytest-asyncio <2 - Async test support, mode `auto` (`pyproject.toml`)
- pytest-cov >=5.0.0 - Coverage reporting (`pyproject.toml`)
- Vitest 4.0 - JavaScript test runner (`web-react/package.json`)
- Testing Library (React 16, DOM 10, jest-dom 6, user-event 14) - React component testing
- jsdom 27 - Browser environment for Vitest (`web-react/vite.config.ts`)
- Playwright - Integration/E2E browser tests (`tox.ini` integration env)
- Vite (rolldown-vite 7.3.1) - Frontend build tool, aliased via `npm:rolldown-vite@7.3.1` (`web-react/package.json`)
- SWC - Fast React compilation via `@vitejs/plugin-react-swc` (`web-react/vite.config.ts`)
- setuptools - Python build backend (`pyproject.toml`)
- tox - Test environment management (`tox.ini`)
- pre-commit - Git hooks for code quality (`.pre-commit-config.yaml`)
- semantic-release - Automated versioning and release (`.releaserc`)
## Key Dependencies
- `mlflow` >=3.10.0, <4 - Core tracking server this plugin extends (`pyproject.toml`)
- `fastapi` >=0.132.0 - ASGI web framework for auth routes (`pyproject.toml`)
- `uvicorn` >=0.41.0 - ASGI server (`pyproject.toml`)
- `authlib` <2 - OIDC/OAuth2 client library, JWT validation (`mlflow_oidc_auth/oauth.py`, `mlflow_oidc_auth/auth.py`)
- `sqlalchemy` >=2.0.46, <3 - ORM for auth database (`mlflow_oidc_auth/sqlalchemy_store.py`)
- `alembic` <2, !=1.18.4 - Database migrations (`mlflow_oidc_auth/db/utils.py`)
- `flask` <4 - MLflow's web layer, hooks registered directly (`mlflow_oidc_auth/app.py`)
- `requests` >=2.32.5, <3 - HTTP client for OIDC discovery/JWKS (`mlflow_oidc_auth/auth.py`)
- `httpx` >=0.28.1 - Async HTTP client (`pyproject.toml`)
- `python-dotenv` <2 - `.env` file loading (`mlflow_oidc_auth/config.py`)
- `asgiref` >=3.11.1 - ASGI utilities (`pyproject.toml`)
- `gunicorn` <24 - WSGI server for non-Windows (`pyproject.toml`, platform conditional)
- `click` - CLI framework for `mlflow-oidc-server` command (`mlflow_oidc_auth/cli.py`, transitive via mlflow)
- `react` 19.1 / `react-dom` 19.1 - UI rendering
- `react-router` 7.9 - Client-side routing (devDependency in package.json but used at runtime)
- `dompurify` 3.3 - HTML sanitization
- `@fortawesome/*` 7.1 / 3.1 - Icon components

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mlflow-oidc/mlflow-oidc-auth](https://github.com/mlflow-oidc/mlflow-oidc-auth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
