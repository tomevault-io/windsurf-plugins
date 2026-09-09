---
trigger: always_on
description: Orientation for AI agents (and new engineers) working in `nerves_hub_web`.
---

# AGENTS.md

Orientation for AI agents (and new engineers) working in `nerves_hub_web`.
It describes the codebase as it exists on `main`.

Keep this file current: when a change alters the layout, tooling, conventions,
or a gotcha below, update the relevant section in the same PR.

## What NervesHub is

NervesHub is the server for managing and deploying firmware to fleets of
[Nerves](https://nerves-project.org/) (embedded Elixir) devices. A device holds
a persistent socket connection to the server; through it the server ships
firmware updates and archives, drives remote console / local-shell sessions,
receives health/geo/log telemetry, and coordinates deployments. The web side is
a Phoenix LiveView dashboard plus a JSON API and a device-facing CLI.

## Runtime shape

A production cluster is **asymmetric**, and this shapes a lot of the code:

- **Device nodes** (many) terminate device socket connections. A given device's
  channels live on whichever device node it connected to.
- **Web nodes** (few) serve the dashboard, API, CLI, and run the deployment
  orchestrators.

Key runtime pieces:

- **Phoenix endpoints** — `NervesHubWeb.Endpoint` (dashboard/API), plus a
  device socket endpoint and a health-check endpoint.
- **Two data stores:**
  - **PostgreSQL** via `NervesHub.Repo` — core domain data (accounts, devices,
    firmware, deployments, …). Migrations in `priv/repo/`.
  - **ClickHouse** via `NervesHub.AnalyticsRepo` — high-volume analytics:
    device connection history, metrics, and the Insights pages. Migrations in
    `priv/analytics_repo/`. Reads/writes are **eventually consistent** (see
    [Testing](#testing)).
- **Background jobs** — Oban (`lib/nerves_hub/workers/`).
- **Deployment orchestration** — one singleton `Orchestrator` process per
  deployment group, owned by ProcessHub (`NervesHub.ManagedDeployments.Distributed`).
- **Presence/liveness** — `NervesHub.Tracker` + Phoenix Presence.
- **Cross-node messaging** — two transports. `Phoenix.PubSub` for dense
  fan-out, and the `group` library for per-entity topics whose consumers are
  sparse (per device, per console session, per product, per firmware). See
  [docs/cross_node_messaging.md](docs/cross_node_messaging.md) for which is
  which and how to choose for something new.

## Repository layout

```
lib/nerves_hub/       Contexts — business logic, no web concerns
lib/nerves_hub_web/   Web layer — endpoints, router, LiveViews, channels, API
test/                 Mirrors lib/; test/support/ has case templates + fixtures
config/               config.exs, runtime.exs, dev.exs, test.exs
priv/repo/            Postgres migrations
priv/analytics_repo/  ClickHouse migrations
assets/               JS/CSS (esbuild + tailwind)
rel/                  Release config
docs/                 Design docs
```

Every environment variable `config/runtime.exs` reads is documented in
[docs/runtime_configuration.md](docs/runtime_configuration.md); keep it in step
when adding or removing one.

### `lib/nerves_hub/` (contexts)

- `accounts.ex` / `accounts/` — users, orgs, org-users, tokens, scopes.
- `devices.ex` / `devices/` — device lifecycle, connections, health status,
  metrics, and network identities (a device's identity on networks NervesHub
  doesn't run, such as iroh or NetBird).
- `managed_deployments.ex` / `managed_deployments/` — deployment groups and the
  `Distributed.Orchestrator` (one per deployment).
- `firmwares.ex` / `firmwares/` and `archives.ex` / `archives/` — firmware and
  archive artifacts, uploads, and firmware **delta** building.
- `products.ex` / `products/` — products and product settings, including
  health profiles (the per-product thresholds behind device health status,
  evaluated in `devices/health_evaluation.ex`).
- `extensions.ex` / `extensions/` — the device **extension framework**
  (`health`, `geo`, `local_shell`, `logging`, `network_identity`,
  `error_reports`); extensions attach per-device and exchange messages over the
  extensions channel.
- `error_reports.ex` / `error_reports/` — exceptions devices report, grouped
  into issues. Split across both stores: the group in Postgres, the
  occurrences in ClickHouse. See [docs/error_reports.md](docs/error_reports.md).
- `scripts.ex` / `scripts/` — support scripts run against a device console.
- `workers/` — Oban workers (e.g. firmware delta building, firmware deletion).
- Cross-cutting: `audit_logs.ex`, `product_notifications.ex`, `tracker.ex`,
  `rate_limit.ex`, `cli_session_cache.ex`, `device_link.ex`, `repo.ex`,
  `analytics_repo.ex`, `application.ex`.

### `lib/nerves_hub_web/` (web)

- `channels/` — the device socket and its channels:
  - `DeviceSocket` (device-side socket; `id/1` is `device_socket:<id>`),
    `DeviceChannel`, `ConsoleChannel`, `ExtensionsChannel`.
  - User-facing counterparts: `UserConsoleChannel`, `UserLocalShellChannel`.
- `live/` — LiveViews (the dashboard). `live/devices/show.ex` is the device
  page; its tabs live in `components/device_page/*` as tab components that hook
  into the LiveView.
- `components/` — shared function/live components.
- `controllers/` — JSON API (`controllers/api/`) and others.
- `plugs/`, `router.ex`, `endpoint.ex`, `device_endpoint.ex`,
  `health_check_endpoint.ex`, `auth*.ex`.

## Tooling & running locally


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nerves-hub/nerves_hub_web](https://github.com/nerves-hub/nerves_hub_web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
