---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

e2a is an authenticated email gateway for AI agents. It provides SMTP relay with SPF/DKIM verification, WebSocket real-time delivery, webhook delivery, a CLI, TypeScript and Python SDKs, and a Next.js web dashboard. Polyglot monorepo: Go (backend), TypeScript (CLI + SDK), Python (SDK), React/Next.js (web).

## Common Commands

### Go backend
```bash
make build              # go build -o bin/e2a ./cmd/e2a
make run                # build + run (uses config.yaml; copy from config.example.yaml first)
make test               # all Go tests (needs Postgres on :5433)
make test-unit          # Go unit tests only (no DB needed)
make test-integration   # Go integration tests (needs Postgres on :5433)
make test-e2e           # Go e2e tests (needs Postgres on :5433)
make docker-up          # start local Postgres + Mailpit via docker compose
make migrate            # apply SQL migrations to local DB
```

Go tests that need the database use `E2A_TEST_DATABASE_URL="postgres://e2a:e2a@localhost:5433/e2a_test?sslmode=disable"`.

**Outbound mail in dev (Mailpit catch-all).** `make docker-up` also starts [Mailpit](https://github.com/axllent/mailpit) — a single-binary SMTP server that captures every outbound message and exposes them at http://localhost:8025. The dockerized `e2a` service points at it automatically. For `make run` (host Go binary), uncomment the Mailpit block in `config.example.yaml`'s `outbound_smtp` section before copying to `config.yaml`, or set `E2A_OUTBOUND_SMTP_HOST=localhost`, `E2A_OUTBOUND_SMTP_PORT=1025`, `E2A_OUTBOUND_SMTP_FROM_DOMAIN=e2a.localhost`. Use this to exercise HITL approval notifications and the `/v1/agents/{email}/test` button locally without real SMTP creds.

### TypeScript SDK & CLI (npm workspaces)
```bash
npm install --package-lock=false           # install all workspace deps
npm run build --workspace @e2a/sdk         # build SDK (must build before CLI)
npm test --workspace @e2a/sdk              # SDK unit tests (vitest)
npm run test:contract --workspace @e2a/sdk # SDK contract tests (needs live server)
npm test --workspace @e2a/cli              # CLI tests (vitest)
npm run build --workspace @e2a/cli         # build CLI
```

### Python SDK
```bash
cd sdks/python
pip install -e ".[dev]"     # install with dev deps
pytest tests/ -v            # unit tests
pytest tests/test_contract.py -v  # contract tests (needs live server)
```

### Web dashboard
```bash
cd web
npm install
npm run dev     # dev server (proxies /api/* to localhost:8080)
npm test        # Jest tests
npm run lint    # ESLint
npm run build   # static export
```

### Code generation
```bash
make spec           # regenerate api/openapi.yaml from the live /v1 Huma handlers
make generate-sdk   # regenerate the TS + Python SDK bases from api/openapi.yaml (OpenAPI Generator)
make generate       # both of the above
```

After changing a `/v1` handler, run `make generate` and commit the regenerated `api/openapi.yaml` plus the SDK bases in `sdks/typescript/src/v1/generated/` and `sdks/python/src/e2a/v1/generated/`. CI (`spec-check` + `generate-sdk-check`) fails if either is stale. (The legacy swag pipeline is gone — `web/public/openapi.yaml` is a frozen copy for the dashboard's API-reference page only and no longer feeds the SDKs.)

## Architecture

### Go backend (`cmd/e2a/` + `internal/`)

The main server (`cmd/e2a/main.go`) runs an SMTP relay and HTTP API. Key internal packages:

- **relay** — SMTP server, receives inbound email
- **emailauth** — SPF/DKIM verification on inbound messages
- **agent** — Agent CRUD, API endpoints, routes
- **identity** — Domain ownership verification and storage
- **headers** — HMAC-SHA256 signing of `X-E2A-Auth-*` headers
- **webhook** — HTTP POST delivery to agent endpoints with retry
- **ws** — WebSocket hub for real-time message push
- **outbound** — Compose and send emails via upstream SMTP (SES)
- **billing** — Stripe integration, usage metering
- **auth** — API key authentication
- **config** — YAML config + env var overrides

Inbound flow: SMTP → emailauth (SPF/DKIM) → agent lookup → headers signing → webhook or WebSocket delivery.

### OpenAPI spec source of truth

The `/v1` surface (`internal/httpapi`, Huma) emits its OpenAPI 3.1 document from
the typed handlers. `make spec` regenerates the committed copy at
`api/openapi.yaml`; `make spec-check` (and `TestSpecGoldenNoDrift`, which runs in
`make test-unit`) is the drift gate — the committed spec must byte-equal what the
live handlers emit, so it can never lag the server. Regenerate + commit
`api/openapi.yaml` after any `/v1` handler change.

### SDK type generation pipeline

The SDK base clients are generated from the canonical Huma spec by OpenAPI
Generator (`openapitools/openapi-generator-cli`), no swag step:

```
api/openapi.yaml (Huma 3.1)
  → openapi-generator (typescript) → sdks/typescript/src/v1/generated/   (the oag base)
  → openapi-generator (python)     → sdks/python/src/e2a/v1/generated/    (package e2a.v1.generated)
```

`make generate-sdk` (= `generate-sdk-ts` + `generate-sdk-py`) regenerates both

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tokencanopy/e2a](https://github.com/tokencanopy/e2a) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
