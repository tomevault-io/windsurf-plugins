---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Chadmin is a ClickHouse administration panel for monitoring running queries and managing user access. It operates in three topology modes — **single-node**, **self-hosted cluster**, and **ClickHouse Cloud** — and auto-detects which one it's in. The UI is a real-time dashboard that groups nodes/services, shows per-node queries and memory, and lets operators kill queries or wake idle Cloud services.

## Architecture

**Backend:** PHP 8.5 / Symfony 8.0 — API endpoints + Twig templates that bootstrap the React frontend. No ORM, no entities, no auth.

**Frontend:** React 19 + TypeScript + Vite 8 + Tailwind 4 — lives entirely in `frontend/`. Uses `@clickhouse/click-ui` (`ClickUIProvider`, `ThemeName`), `highlight.js`, and `sql-formatter`. Built output goes to `public/build/`. Data fetches are plain `fetch` calls; no react-query.

### Two entry points, not a SPA router

- `frontend/index.html` → `main.tsx` → `App.tsx` — Query Monitor at `/`
- `frontend/users.html` → `users.tsx` → `UsersPage.tsx` — Users & Access at `/users`

Each page is its own bundle. Vite is configured to emit deterministic filenames (`assets/index.js`, `assets/users.js`, `assets/style.css`) so Twig can reference them directly without a manifest — see `frontend/vite.config.ts`. If you add a new page, register it in `rollupOptions.input`, add a Twig template that links `/build/assets/<name>.js`, and add a controller route.

### Topology model (`src/Topology/`) — the spine of the app

Everything cluster-aware flows through a single `Topology` built on first request and memoized for the rest of it:

- **`TopologyMode`** — `Single | Cluster | Cloud`.
- **`TopologyDetector`** — picks the mode in this order: `CLICKHOUSE_MODE` env override → cloud credentials present → SQL probe of `system.clusters` (ignoring Cloud's `default` and `all_groups.*`) → single. The SQL probe uses the legacy `ClickHouseClient` (the default connection from env).
- **`TopologyConfig`** — readonly DTO that all env-var reading funnels through, injected into provider/detector/CloudApiClient.
- **`Target`** — a single addressable thing (node/cluster/cloud service) with `hosts`, `port`, `https`, `cluster`, `state`, plus optional Cloud metadata (`warehouseId`, `isPrimary`, `region`, memory limits).
- **`TargetState`** — enum covering Cloud service states (`running/idle/stopped/awaking/...`). `isQueryable()` is true only for `Running | PartiallyRunning | Degraded`.
- **`TopologyProvider::getTopology()`** — returns a `Topology` (list of Targets + org info). In Cloud mode delegates to `CloudTopologyFactory`; if the Cloud API fails, falls back to a one-target "degraded cloud" topology derived from env, with service name parsed from the pod hostname.

### Controller argument resolution

Two value resolvers, both priority 150 in `config/services.yaml`:

- **`TargetResolver`** — resolves a `Target $target` arg from `?target=<id>` (404s if unknown) or falls back to `Topology::primaryTarget()`. **This is the modern path** — new endpoints should take `Target $target`.
- **`ClusterNameResolver`** — legacy: fills `?string $cluster` by querying `system.clusters` directly. Kept only for `/api/get-cluster-name`.

### Data-plane connections (`src/ClickHouse/`)

- **`ConnectionManager::clientFor(Target)`** — builds an `smi2/phpclickhouse` Client per Target, lazy and cached per-request by target id. **Never instantiates for non-queryable targets** (throws `TargetNotQueryableException`) — crucial because even a `ping()` on an idle Cloud service wakes it. For multi-host targets (self-hosted failover) it pings each with a 2s timeout; for single-host targets (Cloud, or a single self-hosted host) it skips the ping entirely and returns the client as-is.
- **`QueryBuilder`** — produces SQL that branches on `Target::mode` / `Target::cluster`:
  - Single-node → `FROM system.<table>` directly.
  - Cluster / Cloud → `FROM clusterAllReplicas('<cluster>', system, <table>)` + `SETTINGS skip_unavailable_shards = 1`.
  - Cloud per-service cluster is always named `'default'`; self-hosted uses the probed/overridden name.
- **`ClickHouseClient`** (`src/Service/ClickHouseClient.php`) — legacy thin wrapper used only by `TopologyDetector` for the cluster-name probe. Controllers should always go through `ConnectionManager`.

### Cloud control plane (`src/Cloud/`)

- **`CloudApiClient`** — HTTP Basic against `https://api.clickhouse.cloud/v1/`. Control-plane only — **calls here never wake services**. Caches the services list for 30s and organizations for 300s via `CacheInterface`. Retries once on 429/5xx with backoff (honors `x-ratelimit-reset`). Unwraps the `{status, requestId, result|error}` envelope.
- **Wake semantics** (`wakeService`): `stopped` → `PATCH /state {"command":"start"}` on control plane; `idle` → GET `/ping` on the data-plane HTTPS endpoint (per Cloud docs, `start` does **not** resume idle services). Read timeouts on the idle-wake ping are expected and swallowed; only connect-level failures bubble up.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bun4uk/chadmin](https://github.com/bun4uk/chadmin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
