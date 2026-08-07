---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Pug is an events ingestion platform built with Go, using PostgreSQL, ClickHouse, and NATS for data storage and messaging. It also provides push notification capabilities (campaigns, delivery, devices).

## Build & Run Commands

```bash
# Build the Go binary
make build

# Run tests
make test

# Start dev infrastructure (PostgreSQL, NATS, ClickHouse)
make infra

# Stop infrastructure
make infra-down

# OTLP collector + HyperDX (optional; for telemetry export)
make clickstack

# Run database migrations
./bin/pug postgres migrate
./bin/pug nats migrate
./bin/pug clickhouse migrate

# Seed the demo project for local dev. Resets Postgres + ClickHouse (pass
# --no-reset to keep the schema and truncate the demo tables instead), then runs
# the same event-gated flow as `pug worker demo`: ensure the demo account,
# backfill events, seed Postgres profiles for ONLY the users that produced
# events, then copy them to ClickHouse. A profile therefore never exists without
# events. (Tunables: --count, --batch.)
./bin/pug seed

# Start development server + workers together
./bin/pug dev

# Start server only
./bin/pug server

# Start individual workers
./bin/pug worker events
./bin/pug worker profile identify
./bin/pug worker profile alias
./bin/pug worker profile upsert

# Rolling demo-traffic generator. Gated by PUG_DEMO_ENABLED everywhere: when off
# (default), `pug dev` skips it and the standalone `pug worker demo` idles (stays
# running but generates nothing, so a k8s Deployment doesn't restart-loop on
# exit); when on, it runs. It derives the demo
# project from the demo user (woof@pug.sh) — creating the customer/org/project
# on a fresh DB, resolving it otherwise — then, if the project has no events
# yet, backfills ~4 months of "Pug & Pals" history and seeds a profile only for
# the users those events belong to (never for a user with no events). Most of
# the pool stays profile-less: the ~half whose join date is still in the future
# (they sign up live as the wall clock crosses their join) plus past users who
# churned before the backfill window. It then plays sessions out in real time.
# Both the backfill and the live stream write straight to ClickHouse via the
# same insert path — the worker owns its ClickHouse connection and uses no NATS,
# so it depends on no other worker (the rollup MV still fires on the direct
# inserts). New signups keep appearing as the wall clock crosses each user's
# join date and the worker creates their profile on first sight.
# Self-bootstrapping so a single k8s
# deployment seeds-then-streams with no manual seed step and no project id to
# configure.
./bin/pug worker demo
```

Environment variables are documented in `.env.example`. **Telemetry export is auto-detected** (decided once on first `SetupSDK` in server/workers): if any standard OTLP endpoint var is set (`OTEL_EXPORTER_OTLP_ENDPOINT`, or a per-signal `OTEL_EXPORTER_OTLP_{TRACES,METRICS,LOGS}_ENDPOINT`), pug exports via OTLP (`otelslog`; needs a collector, e.g. `make clickstack`); otherwise it falls back to application logs as text on stdout with noop trace/metric export (use for deploys without a collector). There is no `PUG_OTEL` switch, and a present-but-blank endpoint counts as unset. Set `OTEL_SERVICE_NAME` when exporting via OTLP.

### Code Generation

```bash
# Generate sqlc queries (after modifying SQL files)
make sqlc

# Generate protobuf code (after modifying .proto files)
# Runs three buf plugins: protoc-gen-go, protoc-gen-connect-go, and
# protoc-gen-mcp (an in-repo wrapper — cmd/protoc-gen-mcp — that adds editions
# support to redpanda-data/protoc-gen-go-mcp, which upstream
# lacks). The MCP plugin emits a `<pkg>mcp` subpackage for every proto package (buf
# has no per-plugin path filter, so every proto is fed to it); only the
# insights/activity/profiles packages are linked into the /mcp endpoint (see MCP
# subsystem), the rest go unused. An RPC's leading comment becomes its MCP tool
# description — for those three services a proto comment is shipped to the model, so
# treat it as runtime behavior. Delete the wrapper and point buf at upstream once it
# declares editions.
make rpc

# Generate templ email templates (after modifying .templ files)
make templ

# Lint Go code
make lint

# Lint proto files
make lint-proto
```

## Architecture

### Backend (Go)

The backend follows a layered architecture with Connect RPC (HTTP/2):

- **`internal/app/`** - CLI entry points using Cobra, split by feature (server, workers, dev, migrate)
  - `server/rpc/` - RPC handlers that map proto services to business logic
  - `workers/campaigns/`, `workers/devices/`, `workers/profiles/`, `workers/events/`, `workers/scheduler/` - NATS message consumers
- **`internal/core/`** - Business logic layer with service and repo per domain (auth, campaigns, delivery, devices, orgs, profiles, projects)
- **`internal/gen/`** - Generated code (do not edit manually)
  - `proto/` - Generated from .proto files via buf
  - `repo/dbread/`, `repo/dbwrite/` - Generated from SQL via sqlc

### Database Pattern

PostgreSQL uses read/write separation:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pug-sh/pug](https://github.com/pug-sh/pug) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
