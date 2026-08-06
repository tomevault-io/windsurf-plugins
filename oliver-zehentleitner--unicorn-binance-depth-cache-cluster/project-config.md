---
trigger: always_on
description: > **End-user cheatsheet for AI-assisted consumption:** [`llms.txt`](llms.txt) — use that one if you're writing code *against* this cluster.
---

# AGENTS.md — UNICORN Binance DepthCache Cluster

> **End-user cheatsheet for AI-assisted consumption:** [`llms.txt`](llms.txt) — use that one if you're writing code *against* this cluster.
> **This file** is for AI agents working *on* this repo itself.

## Why things are the way they are

See [`context/index.md`](context/index.md) before making non-trivial changes — it points to the reasoning behind design decisions, rejected alternatives, and constraints that aren't visible in the code. If `AGENTS.local.md` exists in this repo, that's personal/local notes, not relevant to anyone else.

## Planning & Backlog

Open development tasks and decisions are tracked in **[TASKS.md](TASKS.md)**.

---

## Project Overview

Distributed microservice cluster (MIT License) that manages multiple local Binance order book depth caches. Runs on Kubernetes OR locally via `pip install ubdcc`. Clients interact via a public REST API — no local Binance connection required on the client side.

**Abbreviation:** UBDCC
**Current Version:** 0.9.1
**Author:** Oliver Zehentleitner
**Python:** 3.9-3.14 on Linux, macOS, Windows
**Runtime:** Kubernetes, Docker, or local processes

---

## Package Structure

Five interdependent Python packages under `packages/`:

| Package (PyPI) | Directory | Namespace | Purpose |
|---|---|---|---|
| `ubdcc-shared-modules` | `packages/ubdcc-shared-modules/` | `ubdcc_shared_modules` | Core library: App, Database, REST base, server |
| `ubdcc-mgmt` | `packages/ubdcc-mgmt/` | `ubdcc_mgmt` | Management service: orchestration, node registry, DC distribution |
| `ubdcc-dcn` | `packages/ubdcc-dcn/` | `ubdcc_dcn` | Depth Cache Node: runs UBLDC instances per pod |
| `ubdcc-restapi` | `packages/ubdcc-restapi/` | `ubdcc_restapi` | Public REST API: proxies requests to DCN pods |
| `ubdcc` | `packages/ubdcc/` | `ubdcc` | Meta-package + CLI cluster manager |

**Dependency order:** shared-modules ← mgmt, dcn, restapi ← ubdcc (meta)

The 4 core packages are Cython-compiled. `ubdcc` is pure Python.

---

## Architecture

```
Client (HTTP, any language)
  ↓
ubdcc-restapi  (LoadBalancer, port 80 in K8s / 42081 local)
  ↓ proxies to mgmt for routing info
ubdcc-mgmt     (internal, port 8080 in K8s / 42080 local)
  ├── distributes depth caches to DCN pods
  ├── manages in-memory Database (depthcaches, pods, nodes)
  └── DB replicated to every node via /ubdcc_mgmt_backup (self-healing)

ubdcc-dcn      (multiple, port 8080 in K8s / 42082+ local)
  ├── registers with mgmt on startup
  ├── runs BinanceLocalDepthCacheManager (UBLDC) per DepthCache
  └── reports status to mgmt, holds DB backup
```

**Dev mode:** When K8s config is not available, services run locally with default ports 42080 (mgmt), 42081 (restapi), 42082+ (DCN, auto-increment).

**Recommended sizing:** One DCN per CPU core (GIL limits each process to one core).

---

## Key Files

| File | Purpose |
|---|---|
| `ubdcc_shared_modules/App.py` | Base app class: runtime info, REST server, node registration/sync, shutdown |
| `ubdcc_shared_modules/Database.py` | Thread-safe in-memory store: depthcaches, pods, nodes |
| `ubdcc_shared_modules/RestEndpointsBase.py` | Shared REST helpers: test, shutdown, backup sync, response formatting |
| `ubdcc_shared_modules/ServiceBase.py` | Base class for all three services, port retry on bind failure |
| `ubdcc_mgmt/RestEndpoints.py` | All cluster management endpoints |
| `ubdcc_dcn/DepthCacheNode.py` | Main DCN loop: creates/stops UBLDC instances based on mgmt assignments |
| `ubdcc_restapi/RestEndpoints.py` | Public API: proxies get_asks, get_bids, create_depthcache(s), etc. |
| `ubdcc/cli.py` | CLI cluster manager with interactive `ubdcc>` console |

---

## Public REST API Endpoints (via restapi)

| Endpoint | Method | Description |
|---|---|---|
| `/test` | GET | Health check |
| `/create_depthcache` | GET | Create a single DepthCache |
| `/create_depthcaches` | GET/POST | Create multiple (POST: JSON body; GET: comma-separated markets) |
| `/get_asks` / `/get_bids` | GET | Get order book side (limit_count, threshold_volume) |
| `/get_cluster_info` | GET | Cluster state: pods, nodes, depthcaches |
| `/get_depthcache_list` | GET | List all DepthCaches with status |
| `/get_depthcache_info` | GET | Info for a specific DepthCache |
| `/stop_depthcache` | GET | Stop and remove a DepthCache |
| `/shutdown` | GET | **Dev mode only** — shut down this process |

All accept `debug=true` for timing/routing details.

---

## CLI (ubdcc package)

```bash
pip install ubdcc
ubdcc start --dcn 4          # starts mgmt + restapi + 4 DCN in interactive shell
```

Interactive shell commands: `status`, `add-dcn [count]`, `remove-dcn <count|name>`, `restart <name>`, `stop`, `help`.

`.ubdcc` state file in CWD stores the mgmt port so `status`/`stop` find it without `--port`.

---

## Kubernetes Resources

- **Namespace:** `ubdcc`
- **Manifests:** `admin/k8s/` (StatefulSets, DaemonSet for DCN, Services)
- **Setup:** `admin/k8s/setup/` (namespace, RBAC)
- **Helm chart:** `dev/helm/ubdcc/`

Helm chart and K8s YAMLs reference `ghcr.io` — the OVH-registry migration (tracked in `TASKS.md`) has landed; no OVH references remain in `admin/k8s/` or `dev/helm/ubdcc/`.

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oliver-zehentleitner/unicorn-binance-depth-cache-cluster](https://github.com/oliver-zehentleitner/unicorn-binance-depth-cache-cluster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
