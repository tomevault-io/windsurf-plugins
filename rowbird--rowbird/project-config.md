---
trigger: always_on
description: Guidance for AI coding agents (and humans) working on this repository. Read
---

# AGENTS.md: Rowbird

Guidance for AI coding agents (and humans) working on this repository. Read
[CONTRIBUTING.md](CONTRIBUTING.md) too.

Rowbird is a self-hosted, open source service that runs SQL queries on a schedule and delivers the
results, formatted as CSV, Excel, PDF, JSON or inline tables, to email, Telegram, Slack, Discord,
webhooks, Uptime Kuma, S3 and more. Conditions turn reports into data alerts.

Tagline: **"Your SQL results, delivered."** · Website: https://rowbird.dev · Docs: https://docs.rowbird.dev

## Writing style

- **Everything in the repository is in English**: code, identifiers, comments, commit messages,
  docs, ADRs, OpenAPI descriptions, log messages and error codes. The only exception is the content
  of the `pt-BR` locale files.
- **Write in natural, clear prose**, like a senior engineer explaining something to a colleague.
- **No emojis**, anywhere (code, comments, commits, docs).
- **No em dashes or en dashes** (the long dash characters). Use commas, colons, parentheses or
  separate sentences instead. Regular hyphens in compound words and code are fine.
- Avoid filler, hype and excessive bold. Be direct and specific.

---

## Sources of truth (read before working)

| What | Where |
|---|---|
| Product & behavior spec | `docs/spec/*.md` (one file per topic) |
| Architecture decisions | `docs/adr/*.md` (do NOT silently contradict an accepted ADR) |
| Roadmap | `docs/plan/ROADMAP.md` (what ships in v1 and what comes next) |
| HTTP API contract | `api/openapi.yaml` (spec-first, everything else is generated from it) |

If the spec is ambiguous, prefer the simplest behavior consistent with it and **write down the
decision** (update the spec, or add an ADR if it is architectural). If the spec seems wrong, stop and
ask instead of improvising.

## Stack

- **Backend:** Go (latest stable), single binary. Router `chi`, logging `log/slog`, OpenAPI server
  stubs via `oapi-codegen` (strict server).
- **Internal store:** SQLite by default (`modernc.org/sqlite`, pure Go, no CGO), PostgreSQL optional
  (`pgx`). Migrations per dialect. Every table carries `workspace_id`.
- **Frontend:** Vue 3 + TypeScript + Vite, `<script setup lang="ts">`, Pinia, Vue Router, vue-i18n,
  Tailwind + shadcn-vue, CodeMirror 6 for SQL. API client generated with `openapi-typescript` +
  `openapi-fetch`. Built assets are embedded in the Go binary via `embed`.
- **Key libs:** excelize (XLSX), maroto v2 (PDF), robfig/cron v3 (cron parsing only), minio-go (S3),
  x/crypto (argon2id, ssh), pquerna/otp (TOTP), go-webauthn (passkeys), coreos/go-oidc,
  prometheus/client_golang, a Mustache implementation for message templates.
- **Tests:** Go `testing` + testcontainers-go; Vitest; Playwright. **Docs:** VitePress. **Release:** GoReleaser.

## Repository layout

```
cmd/rowbird/            main.go: CLI entrypoint (serve, migrate, backup, apply, ...)
api/openapi.yaml        API contract (source of truth)
internal/
  api/                  HTTP handlers implementing generated interfaces, middleware
  app/                  wiring / dependency injection
  auth/                 sessions, passwords, TOTP, passkeys, OIDC, API keys
  store/                repositories + migrations (sqlite/, postgres/); workspace scoping lives here
  scheduler/            due-report polling, claiming, next_run_at
  runner/               worker pool, run lifecycle, retries
  plugin/               registry + shared contracts (Connector, Formatter, Condition, Destination, ...)
  connector/<driver>/   postgres, mysql, mssql, sqlite (+ ssh tunnel helper)
  format/<name>/        csv, xlsx, json, pdf, inline (html, markdown, text)
  condition/            built-in conditions
  destination/<name>/   email, telegram, slack, discord, webhook, uptimekuma, s3
  storage/<name>/       artifact storage: local, s3
  ai/<provider>/        openai, anthropic, gemini, ollama, openai-compatible
  notify/               in-app notifications, system alerts, grouping, heartbeat
  gitops/               YAML config-as-code: export, import, apply, diff
  crypto/               AES-256-GCM secret encryption, key rotation
  params/               query parameter parsing + binding
  i18n/                 server-side message catalogs
  metrics/  health/  security/
ee/                     commercial-licensed code (empty in v1, reserved)
web/                    Vue app
docs/                   spec, adr, plan (roadmap), site (VitePress, docs.rowbird.dev), assets
deploy/                 docker-compose examples, systemd unit, k8s example manifests
testdata/               fixtures, demo seed SQL
```

## Commands (maintain these in the Makefile)

```
make dev               # backend with live reload + Vite dev server (proxy /api)
make generate          # regenerate Go server stubs + TS client from api/openapi.yaml
make test              # unit tests (Go + Vitest)
make test-integration  # testcontainers: postgres, mysql, mariadb, mssql, mailpit, versitygw (S3)
make test-e2e          # Playwright against a built binary (Docker: mailpit, versitygw)
make lint              # golangci-lint + eslint + vue-tsc
make build             # web build -> embed -> single binary in ./bin/rowbird
```

## How to work


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rowbird/rowbird](https://github.com/rowbird/rowbird) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
