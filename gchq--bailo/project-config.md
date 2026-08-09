---
trigger: always_on
description: Bailo is a full-stack monorepo for managing the lifecycle of machine learning models - supporting scalability, impact,
---

# Bailo Agent Development Guide

## Project

Bailo is a full-stack monorepo for managing the lifecycle of machine learning models - supporting scalability, impact,
collaboration, compliance and sharing. The stack is a TypeScript/Node.js backend (Express), a Next.js/React frontend, a
Python client library, and a Python FastAPI scanning service.

Core principles for changes:

- Always use British English for spellings and terms (e.g. "colour", "organisation", "authorise", "centre", "licence").
- Prefer clear, readable code over clever or compact solutions. Keep functions focused on a single responsibility.
- Avoid new external dependencies unless absolutely necessary. Reuse existing project utilities first.
- Backend follows a layered architecture: routes -> services -> models -> connectors. Respect this separation.
  Connectors use a strategy pattern: a base class in `Base.ts`, concrete implementations (e.g. `clamAv.ts`, `oauth.ts`),
  and an `index.ts` that selects the implementation via `config.connectors.<name>.kind`. Categories: authentication,
  authorisation, audit, artefactScanning, metrics, peer.

## Structure

- **`.`** - monorepo root. Orchestrates backend and frontend builds/tests/lint. Docker Compose quickstart:
  `npm i && docker compose build --parallel && docker compose up -d`.
- **`backend`** - Express 5 API server (TypeScript ESM). Mongoose for MongoDB, Zod for validation, OpenTelemetry for
  observability.
- **`frontend`** - Next.js web application (TypeScript, React, MUI). SWR for data fetching.
- **`lib/landing`** - static Next.js site for GitHub Pages. Run `npm run generate` before `dev` or `build`.
- **`lib/python`** - Python client library (`bailo` on PyPI). Partial API coverage. Has unit and integration tests.
- **`lib/artefactscan_api`** - FastAPI scanning service (ModelScan, Trivy, ClamAV) on port 3311.
- **`infrastructure`** - Helm charts and deployment configuration.

## Commands

Bailo expects Node.js 26 (see `.nvmrc`).

### Root (orchestrates backend + frontend)

- Install all dependencies: `npm i` (also installs backend + frontend via postinstall)
- Build: `npm run build`
- Test: `npm run test`
- Lint: `npm run lint`
- Format check: `npm run check-style`
- Format fix: `npm run style`
- Generate certs: `npm run certs`
- Cypress E2E (open): `npm run cy:open`
- Cypress E2E (headless): `npm run cy:run`

### Backend (`backend/`)

- Dev server: `npm run dev`
- Build: `npm run build`
- Test: `npm run test`
- Lint: `npm run lint`
- Format check: `npm run check-style`
- Run a script: `npm run script`
- Seed database: `npm run seed`

### Frontend (`frontend/`)

- Dev server: `npm run dev`
- Build: `npm run build`
- Test: `npm run test`
- Lint: `npm run lint`
- Format check: `npm run check-style`
- Cypress E2E (open): `npm run cy:open`
- Cypress E2E (headless): `npm run cy:run`

### Python client (`lib/python/`)

```bash
python3 -m venv libpythonvenv && source libpythonvenv/bin/activate
pip install -e .[test]
```

- Unit tests: `pytest`
- Integration tests: `pytest -m integration` (requires Bailo running on `https://localhost:8080`)
- MLFlow tests: `pytest -m mlflow`
- Format check: `black --check .`
- Lint: `pylint bailo`

### ArtefactScan API (`lib/artefactscan_api/`)

```bash
python3 -m venv artefactscanvenv && source artefactscanvenv/bin/activate
pip install -r requirements-dev.txt
```

- Unit tests: `pytest`
- Integration tests: `pytest -m integration`
- Docker build & run:
  `docker build -t artefactscan_rest_api:latest . && docker run -p 0.0.0.0:3311:3311 artefactscan_rest_api:latest`

## Coding conventions

### TypeScript/JavaScript

- **Prettier** for formatting: `printWidth: 120`, no semicolons, single quotes, 2-space indent.
- **ESLint** with zero warnings allowed (`--max-warnings=0`).
- ESM imports with `.js` extensions (backend is `"type": "module"`).
- Use **Zod** schemas for request/response validation. Route files export a schema (e.g. `postModelSchema`) used for
  both validation and OpenAPI generation via `@asteasolutions/zod-to-openapi`.
- Use existing error helpers from `backend/src/utils/error.ts`: `BadReq`, `Forbidden`, `NotFound`, `Unauthorized`,
  `Conflict`, `ContentTooLarge`, `UnsatisfiableRange`.
- Prefer self-documenting code. Only add a docstring when the function name, parameters, and types do not make the
  intent obvious. When a docstring is needed, use **JSDoc-style** and keep it to 1-2 lines.

### Python

- **Black** for formatting (line-length 120). Do not manually override.
- **pylint** for linting. Follow **PEP 8** where not overridden by Black.
- Use **reStructuredText (reST)** format for docstrings.
- Add inline comments only where the code is non-obvious.

### Punctuation and characters

- Use hyphens (`-`) instead of em dashes or en dashes.
- Use straight double quotes (`"`) instead of curly quotes.
- Use straight apostrophes (`'`) instead of curly apostrophes.
- Use `->` instead of arrow characters (`→`).
- Check copied text from external sources for special characters before committing.

## Testing

Bug fixes require regression tests that fail before the fix and pass after it. New features require tests covering edge
cases and invalid input.

### Backend (Vitest)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gchq/Bailo](https://github.com/gchq/Bailo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
