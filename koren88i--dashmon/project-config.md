---
trigger: always_on
description: This file provides guidance to OpenAI Codex when working with code in this repository.
---

# AGENTS.md

This file provides guidance to OpenAI Codex when working with code in this repository.

## What this project is

Dashboard SRE — a service that takes a Grafana dashboard JSON and produces:
1. A **probe engine** that actively monitors that dashboard's health
2. A **meta-dashboard** (Grafana JSON) — "the dashboard for the dashboard"
3. **Alert rules** (YAML) for each detectable failure mode

We monitor the **user experience of a specific dashboard** — detecting when a customer would open it and see something wrong (blank panels, stale data, slow loads, broken variables). We do NOT monitor the Grafana stack itself.

The repo also ships a **self-contained demo** with a mock Prometheus backend, fault injection, and a UI simulator.

## Tech stack

- **Python 3.11+**, **FastAPI** (probe engine + mock backends)
- **Playwright** (optional browser render probe)
- **Single-file HTML/JS** for UI simulator (no build step)
- **Docker Compose** — everything runs with `docker compose up`
- **Prometheus exposition format** for probe metrics (`/metrics`)
- **Grafana Alerting YAML** (Grafana 9+ provisioning format)
- Config: YAML for probe config, JSON for dashboard input/output

## Commands

```bash
# Run everything (Docker)
docker compose up --build

# UI simulator (after compose up)
open http://localhost:8080/simulator.html

# Probe engine health endpoint (JSON summary)
curl http://localhost:8000/health

# Probe engine metrics (Prometheus format)
curl http://localhost:8000/metrics

# Mock backend health
curl http://localhost:9090/-/healthy

# Inject a fault (docker or local)
curl -s -X POST http://localhost:9090/faults/inject \
  -H "Content-Type: application/json" \
  -d '{"type":"no_data","target":"http_requests_total","duration_seconds":60}'

# Clear all faults
curl -s -X POST http://localhost:9090/faults/clear \
  -H "Content-Type: application/json" -d '{"target":"all"}'
```

## Ports

| Port | Service |
|---|---|
| 3000 | Grafana (anonymous auth, provisioned dashboards + alerts) |
| 8080 | Demo UI simulator (nginx) |
| 8000 | Probe engine (`/health`, `/metrics`) |
| 9091 | Real Prometheus (scrapes probe engine) |
| 9090 | Mock Prometheus backend + fault injection |
| 8012 | Browser render probe (`/health`, `/metrics`) |

Override via `.env` (copy from `.env.example`): `GRAFANA_PORT`, `PROMETHEUS_PORT`, `SIMULATOR_PORT`, `PROBE_ENGINE_PORT`, `MOCK_BACKEND_PORT`.

## Architecture

```
Grafana Dashboard JSON
        │
        ▼
   parser.py ──→ ProbeSpec objects
        │
        ├──→ engine.py (probe loop, 30s interval, concurrent)
        │       │
        │       ├──→ query_probe.py      (NO_DATA, QUERY_TIMEOUT, SLOW_QUERY, PANEL_ERROR)
        │       ├──→ staleness_probe.py  (STALE_DATA)
        │       ├──→ variable_probe.py   (VAR_RESOLUTION_FAIL)
        │       └──→ cardinality_probe.py(CARDINALITY_SPIKE, METRIC_RENAME)
        │       │
        │       ▼
        │   metrics.py ──→ /metrics (Prometheus) + /health (JSON for UI)
        │
        ├──→ meta_dashboard.py ──→ Grafana dashboard JSON (importable)
        └──→ alert_rules.py    ──→ Grafana alerting YAML (provisionable)
```

**mock_backend/** — FastAPI app mimicking Prometheus HTTP API with fault injection (`POST /faults/inject`, `POST /faults/clear`, `GET /faults/active`). The probe engine talks to it the same way it would talk to real Prometheus.

**demo/simulator.html** — single HTML file. Top: target dashboard with live sparklines. Middle: meta-dashboard polling `/health`. Bottom: fault injection buttons + issue log. Faults must be detected within ≤30s.

## Key data structures

- `PanelProbeSpec` — panel_id, panel_title, datasource_uid, datasource_type, queries (raw PromQL), expected_min_series
- `VariableProbeSpec` — name, datasource_uid, query, is_chained, chain_depth

## Failure modes detected

`NO_DATA`, `STALE_DATA`, `METRIC_RENAME`, `QUERY_TIMEOUT`, `VAR_RESOLUTION_FAIL`, `SLOW_QUERY`, `SLOW_DASHBOARD`, `CARDINALITY_SPIKE`, `PANEL_ERROR`

## Design constraints (do not change)

- Local/unit fixtures may run without Grafana, but Docker probe configs must exercise Grafana's panel request path through `/api/ds/query`
- Probe engine is datasource-agnostic (Prometheus HTTP API; other types out of scope but architecture allows adding them)
- Meta-dashboard JSON must be importable into real Grafana
- Alert rules YAML must be valid Grafana Alerting provisioning format
- UI simulator works in modern browser with no build step
- Docker Compose must work on Mac and Linux
- Probe engine handles errors gracefully — one panel failure doesn't crash others
- No external databases, no heavy frameworks
- Mock APIs must implement the spec, not just the subset our own code uses. Real consumers (e.g. Grafana) will exercise different parts of the contract (e.g. POST instead of GET). If we only test against our own code, we miss what breaks for everyone else.
- A green raw datasource probe is not enough to claim a Grafana dashboard is healthy. User-facing dashboard contracts must include the Grafana datasource/plugin path, because variables, range queries, plugin serialization, and non-JSON errors can fail after Prometheus itself succeeds.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [koren88i/dashmon](https://github.com/koren88i/dashmon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
