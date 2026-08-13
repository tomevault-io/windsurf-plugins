---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Actual-sync** is a self-hosted Node.js service that automates bank transaction synchronization for [Actual Budget](https://actualbudget.org/) servers. It supports multiple budget instances, encrypted budgets, scheduling, multi-channel notifications, and a web dashboard with Prometheus metrics.

## Commands

```bash
# Install git hooks (run once after cloning — also runs automatically via npm install)
git config core.hooksPath .githooks
# This installs a pre-push hook that runs npm test --coverage --ci before every push.

# Run all tests
npm test

# Run a single test file
npm test -- configLoader.test.js

# Run tests matching a name pattern
npm test -- --testNamePattern="should validate configuration"

# Watch mode for a specific file
npm run test:watch -- syncService.test.js

# Generate coverage report
npm run test:coverage

# Check for dead code (unused files/exports) with knip
npm run dead:check     # blocking (exit 1 on findings); CI runs this in the lint job
npm run knip           # report-only (always exit 0); for local diffing

# Start the scheduled sync service
npm start

# Force immediate sync (all servers)
npm run sync

# Sync a specific server
npm run sync -- --server "ServerName"

# Validate config against schema
npm run validate-config

# List discovered bank accounts
npm run list-accounts

# View sync history
npm run history

# Regenerate docs/screenshots/* (Puppeteer, dev-only) and README metric badges
npm run screenshots
npm run badges:generate

# Bump version (updates VERSION + package.json + package-lock.json in sync;
# aborts if local version is behind the latest released tag). For a patch
# release you do not need this (the auto-release patch-bumps). Run it on
# `development` only to cut a MINOR or MAJOR release, which the auto-release
# then publishes as-is (see Git Workflow).
npm run version:bump -- minor   # or: patch / major
```

No build step — this is plain JavaScript (no TypeScript, no bundler).

### Docker Development

```bash
# Build image
docker build -t actual-sync:dev .

# Run locally with volume mounts
docker run --rm \
  -v ./config:/app/config:ro \
  -v ./data:/app/data \
  -v ./logs:/app/logs \
  -e PUID=1001 -e PGID=1001 \
  actual-sync:dev
```

**PUID/PGID & privilege drop:** the container starts as root via `docker/entrypoint.sh`, aligns its user to `PUID`/`PGID` (default `1001:1001`), chowns `/app/data` + `/app/logs`, then drops to that non-root user with `su-exec` under `tini`. Set `PUID`/`PGID` to match the owner of the mounted volumes (Unraid: `99`/`100`); otherwise the budget SQLite DB can't be written and sync fails with "No budget file is open". The published image is built with `npm ci --omit=dev`, so it contains **no** devDependencies. Multi-arch (arm64) images are built only on `main`/`v*` tags; `development` builds amd64 only.

### NAS / server deployment (pull pre-built image)

In deployments that consume the pre-built GHCR image (no `build:` key in the compose file), **`docker compose build` is a no-op** — update by pulling the newly published image:

```bash
docker compose pull actual-sync && docker compose up -d actual-sync
```

## Architecture

### Core Data Flow

```
Scheduler (node-schedule) or manual trigger
  → syncAllBanks() → syncBank(server) [per server]
    → Correlation ID assigned
    → actual.init() → actual.downloadBudget() → actual.getAccounts()
    → actual.sync() [initial state sync]
    → bankSync per account with runWithRetries() [exponential backoff]
    → actual.sync() [final state sync]
    → Results tracked in SQLite + Prometheus
    → Thresholds evaluated → Notifications dispatched
    → actual.shutdown()
```

### Service Initialization (Dependency Injection)

`index.js` validates startup, then `src/syncService.js` wires all services together:

- `lib/configLoader.js` — AJV schema validation against `config/config.schema.json`
- `lib/logger.js` — Custom structured logger (no Winston/Pino); supports file rotation and correlation IDs
- `services/syncHistory.js` — SQLite-backed sync history via `better-sqlite3`
- `services/healthCheck.js` — Express HTTP server. Public (no auth): `/health`, `/ready`, `/metrics`, `/metrics/prometheus`, `/icon.png`, and a WebSocket log stream at `/ws/logs`. Behind `dashboardAuth()`: `/dashboard` and the `/api/dashboard/*` REST API (`status`, `servers`, `orphaned-servers`, `schedules`, `metrics`, `history`, `accounts`, plus POST `sync`, `dismiss-error`, `reset-history`, `test-notification`). A global `express-rate-limit` (60 req/min/IP) covers every route except `/icon.png`, which is exempt so the dashboard's favicon/logo fetch doesn't eat the API budget (#113)
- `services/prometheusService.js` — Prometheus metrics via `prom-client`
- `services/notificationService.js` — Routes alerts to Email, Telegram, ntfy, and webhooks (Slack / Discord / generic). Config keys under `notifications`: `email`, `telegram`, `ntfy`, `webhooks`, plus `branding`, `notifyOnSuccess`, `thresholds`, `rateLimit`. Note there is **no** Teams channel — see the advertised-channels drift guard below before adding one to the README.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agigante80/Actual-sync](https://github.com/agigante80/Actual-sync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
