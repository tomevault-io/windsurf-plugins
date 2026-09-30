---
trigger: always_on
description: - This is a multi-package repository; each Python package has its own Poetry environment and lockfile.
---

# Howler Agent Guide

## Repository Shape

- This is a multi-package repository; each Python package has its own Poetry environment and lockfile.
- `api/` is the Flask backend and owns the shared ODM, datastore, API tests, and local dependency stack.
- `ui/` is the React/Vite TypeScript frontend; its generated entity types live in `ui/src/models/entities/generated/`.
- `client/` is the Python SDK and its integration tests use a running Howler API plus Elasticsearch and Redis.
- `plugins/evidence`, `plugins/sentinel`, and `plugins/sync` are separate Python packages that depend on the API package.
- `mcp/` is the authenticated FastMCP server; unit tests are isolated, while network tests need the full local stack.
- `documentation/` is the MkDocs site. `docs/RELEASES.md` is the product changelog.

## Toolchains And Setup

- Use Poetry for every Python package; do not install Python dependencies from the repository root.
- The API quality environment is Python 3.12 (`api/.python-version`); CI also tests the API, client, and plugins on Python 3.10-3.13 as applicable.
- The UI's `.nvmrc` says Node 20, while current UI CI uses Node 24 and pnpm 11. Use pnpm, never npm, and honor `ui/pnpm-lock.yaml`.
- After changing a lockfile or dependency declaration, reinstall that package with its lockfile before validating it.

## API Development And Validation

Run these from `api/` after `poetry install --with test,dev,types`:

```bash
poetry run ruff format howler --diff
poetry run ruff check howler --output-format=github
poetry run type_check
poetry run pyright --project pyproject.toml --level warning
poetry run test
```

- API tests expect writable `/etc/howler/conf`, `/etc/howler/lookups`, and `/var/log/howler` directories. Copy `build_scripts/classification.yml`, `build_scripts/mappings.yml`, and `test/unit/config.yml` to the corresponding config paths, then run `poetry run mitre /etc/howler/lookups` and `poetry run sigma` before testing.
- Start local dependencies with `docker compose -f api/dev/docker-compose.yml up --build -d`; Elasticsearch, Redis, Kibana, and Keycloak are defined there. `poetry run python build_scripts/docker_health.py` checks their health.
- Run the backend with `poetry run server`.
- The test wrapper starts a temporary test API and forwards pytest arguments, so a focused test can use `poetry run test test/unit/services/test_case_service.py -k rule`.
- `api/build_scripts/generate_classes.py` generates `ui/src/models/entities/generated/`; do not edit those files manually. Generation requires a running API at `localhost:5000` with suitable randomized data.

## UI Development And Validation

Run these from `ui/`:

```bash
pnpm install --frozen-lockfile
python ../hooks/check_translation.py
python ../hooks/find_bad_imports.py
pnpm tsc --noEmit
pnpm oxfmt src --check
pnpm oxlint src
pnpm test
```

- `pnpm run lint` fixes files; use the read-only commands above when checking a change.
- Use `pnpm exec vitest run path/to/file.test.tsx` for a focused test. The default Vitest and oxlint scopes exclude `src/commons/**`; pass a `src/commons/...` path explicitly to test it.
- `pnpm build` performs the production build; `pnpm start` runs Vite on port 3000 and proxies `/api` and `/socket` according to `VITE_API_TARGET`.
- Testing Library is configured with `testIdAttribute: 'id'`; elements targeted by `getByTestId`/`queryByTestId` need an `id`, not `data-testid`.
- Use the mock matching the package imported by the component: router hooks here come from `react-router`, not `react-router-dom`. Await `userEvent` directly rather than wrapping it in `act`; flush fake timers inside `act` with `vi.advanceTimersByTimeAsync`.
- `MockLocalStorage` defines a non-writable `key` method; do not use `key` as a storage key in tests. Clear the mock and its spies in `beforeEach` when using `setupLocalStorageMock()`.
- Import the explicit `@fontsource/roboto/index.css` entry; the package-root side-effect import can fail TypeScript resolution.
- UI lint requires type-only imports and prefers expression/arrow-style functions. Keep braces around control-flow bodies.

## Client, Plugins, And MCP

- Client: from `client/`, run `poetry install --with dev,test,types`, then `poetry run ruff format howler_client --diff`, `poetry run ruff check howler_client --output-format=github`, `poetry run type_check`, and `poetry run test`. A focused test is `poetry run test test/unit/test_case.py -k name`.
- Client integration tests need the API environment configured as above, an API server, and `docker compose -f client/test/docker-compose.yml up -d` for Elasticsearch and Redis. CI seeds API data with `poetry run python -m howler.odm.random_data`.
- Evidence and Sentinel: from the plugin directory, install with `poetry install --with dev`, run Ruff against `evidence` or `sentinel`, run `poetry run type_check`, and use direct pytest for tests, for example `poetry run pytest test/unit -vv`. Their tests import the API package and may need the API config files and services.
- Sync: install from `plugins/sync/` with `poetry install --with dev`; unit tests are local, but integration tests require Elasticsearch and create/clean randomized Howler users and hits.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CybercentreCanada/howler](https://github.com/CybercentreCanada/howler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
