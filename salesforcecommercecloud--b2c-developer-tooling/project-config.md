---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`salesforce-b2c-tooling-sdk` (import name `b2c_tooling_sdk`) is a Python SDK for
Salesforce B2C Commerce tooling: authentication, config resolution, typed
OCAPI/SCAPI/WebDAV clients, and higher-level operations (code deploy, jobs,
sites, catalogs, BM users/roles, sandboxes/ODS, metrics, logs). It lives in the
`python/b2c-tooling-sdk/` subfolder of the larger `b2c-developer-tooling`
monorepo (alongside `python/samples/`), whose root
`CLAUDE.md`/`AGENTS.md` covers the **TypeScript** packages — that guidance
mostly does **not** apply here (no pnpm, no oclif), with one exception:
**versioning uses the root Changesets flow** (see Releasing below and the root
`CLAUDE.md`), since this package is registered as a private, unpublished
`package.json` purely so Changesets can bump its version and changelog.

**This is a faithful port of the TypeScript `@salesforce/b2c-tooling-sdk`**
(`../../packages/b2c-tooling-sdk/`). Parity is the entire point of the project:

- The Python SDK shares the **same on-disk state** as the TS CLI byte-for-byte —
  the persisted auth-session store (`auth-sessions.json` in the oclif data dir
  for app `@salesforce/b2c-cli`) and the same config files (`dw.json`,
  `~/.mobify`, `settings.json`). A token minted by `b2c auth login` must work
  from Python, and vice versa.
- **When porting or changing a module, read the TS source first** — the files
  under `../../packages/b2c-tooling-sdk/src/` are the source of truth. Concepts,
  module layout, and public surface mirror the TS library; only the syntax is
  Pythonic (`async`/`await`, dataclasses, snake_case).

## Commands

All commands run from `python/b2c-tooling-sdk/`. Use the Makefile targets; the `*-agent`
variants produce condensed output for coding agents.

```bash
make install          # pip install -e ".[dev,docs]" (do this in a venv)

make test-agent       # pytest, quiet (failures + short summary only)
make test             # full run with coverage
.venv/bin/pytest tests/test_oauth.py            # a single test file
.venv/bin/pytest tests/test_oauth.py::test_name # a single test

make lint-agent       # ruff check --quiet .
make typecheck-agent  # mypy, single-line errors
make format           # ruff format .
make format-check     # ruff format --check .
```

**The gate** (must be green before committing): `ruff check`, `ruff format
--check`, `mypy` (strict), `pytest`.

```bash
make generate-models  # regenerate Pydantic models from the TS package's specs
make api-docs         # regenerate the API reference pages (../../docs/python/api)
```

## Documentation

Guides live in the monorepo docs site at `../../docs/python/*.md` (VitePress,
served under `/python/`). The API reference in `../../docs/python/api/` is
**generated** from docstrings by `scripts/generate_api_docs.py` (griffe, sphinx
docstring style) and committed; CI runs `make api-docs-check`. After changing a
public docstring or a barrel's `__all__`, run `make api-docs` and commit the
result. New public subpackages must be added to `PAGES` in that script.

## Architecture

Layers, roughly bottom-up (each subpackage has an `__init__.py` barrel that
mirrors the corresponding TS `index.ts`):

- **`auth/`** — OAuth client-credentials, JWT Bearer, PKCE interactive,
  implicit, Basic, API-key strategies + `resolve_auth_strategy`. A module-level
  token cache with single-flight semantics (reset in tests via
  `reset_oauth_cache_for_testing()`). The persistent session store serializes
  snake_case fields back to the TS camelCase JSON keys via an explicit map.
- **`config/`** — `resolve_config` reads `dw.json` (incl. multi-config `configs`
  aliases), `~/.mobify`, `settings.json`. The raw on-disk layer stays camelCase
  (TS parity); `NormalizedConfig` is a snake_case dataclass. Produces a resolved
  config whose `.create_b2c_instance()` method builds a `B2CInstance`.
- **`instance/`** — `B2CInstance` combines instance config + auth to expose lazy,
  typed clients (`.webdav`, `.ocapi`, SCAPI client config).
- **`clients/`** — typed OCAPI, WebDAV, and SCAPI Admin clients.
  **openapi-fetch semantics: clients return `ClientResult(data, error,
  response)` and never raise on 4xx/5xx** — only a network failure raises
  (`NetworkError`). `clients/models/` holds the **generated** Pydantic v2 models.
- **`operations/`** — task-oriented, **success-or-raise** verbs built on top of
  the clients (the opposite error contract from `clients/`). One subpackage per
  domain (`code`, `jobs`, `sites`, `catalogs`, `bm_users`, `bm_roles`, `ods`,
  `metrics`, `logs`, `users`, `roles`, `orgs`). Some domains have a dual-backend
  factory that picks SCAPI or falls back to OCAPI.
- **`slas/`** — SLAS Shopper Login (guest/registered tokens, PKCE helpers).
  Subpath-only: `from b2c_tooling_sdk.slas import ...` (not in the top-level barrel).
- **`sync/`** — see below.

### Async source, runtime sync facade

The SDK is **async-first**. Every public callable also has a blocking twin under
`b2c_tooling_sdk.sync` with an identical signature minus `await`.

**The sync layer is a runtime facade, not generated code.** `sync/_runner.py`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SalesforceCommerceCloud/b2c-developer-tooling](https://github.com/SalesforceCommerceCloud/b2c-developer-tooling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
