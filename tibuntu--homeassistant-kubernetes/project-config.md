---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Home Assistant custom integration for monitoring and controlling Kubernetes clusters. Python 3.13+, async throughout, uses the official `kubernetes` Python client.

## General Instructions

Always update this CLAUDE.md automatically whenever new changes are implemented, so it stays in sync with the current state of the codebase, architecture, and conventions.

Never add yourself as a git co-author. Do not include `Co-Authored-By` trailers in any commit message.

## Commands

```bash
# Install Python dev dependencies
pip install -e ".[dev]"

# Install frontend dependencies and build
cd frontend && npm install && npm run build

# Run all tests (includes coverage)
pytest

# Run a single test file or specific test
pytest tests/test_sensors.py
pytest -k test_specific_name

# Run by marker
pytest -m unit
pytest -m integration
pytest -m "not slow"

# Lint and format
ruff check .
ruff format .

# Type checking
mypy custom_components/

# Security scanning
bandit -c pyproject.toml -r custom_components/

# Pre-commit (runs all checks)
pre-commit run --all-files

# Documentation
zensical serve
```

## Architecture

### Data Flow

```
KubernetesClient (API calls) → KubernetesDataCoordinator (polling/caching) → Entities (sensors/switches/binary_sensors)
```

When the experimental **Watch API** is enabled:
```
KubernetesClient.watch_stream() → KubernetesDataCoordinator._run_watch_loop() → coordinator.data updated + async_update_listeners()
```

**Sidebar panel:**
```
Browser (kubernetes-panel) → hass.callWS() → websocket_api.py → KubernetesDataCoordinator
```

**Entry point:** `__init__.py` sets up the integration lifecycle. On config entry setup, it creates a `KubernetesClient`, wraps it in a `KubernetesDataCoordinator`, then forwards setup to three platforms: `sensor`, `switch`, `binary_sensor`. On the first config entry, it also registers HA services, WebSocket commands (via `async_setup`), and the sidebar panel. If the `enable_watch` option is set, watch tasks are also started after the first refresh.

### Key Modules

- **`kubernetes_client.py`** — Async wrapper around the k8s Python client. Fetches deployments, statefulsets, daemonsets, cronjobs, jobs, pods, nodes. Uses generic `_fetch_resource_list(api_path, resource_name, parse_fn)` and `_fetch_resource_count(api_path, resource_name)` methods to eliminate per-resource boilerplate. Handles SSL (`ssl=self.verify_ssl`), auth, namespace filtering (per-namespace loop vs cluster-wide), and error deduplication (5-min cooldown). Also provides `watch_stream()` (async generator), `list_resource_with_version()`, `ResourceVersionExpired` exception, single-item parse helpers (`_parse_pod_item`, `_parse_node_item`, `_parse_replica_workload_item` aliased as `_parse_deployment_item`/`_parse_statefulset_item`, `_parse_daemonset_item`) used by the coordinator watch loop, `_enrich_workloads_with_metrics(workloads, workload_type_label)` for deployment/statefulset metrics enrichment, `get_node_metrics()` for real-time CPU/memory usage from the Metrics API, `delete_pod(pod_name, namespace)` for pod deletion (aiohttp primary + official client fallback), and `rollout_restart_deployment/statefulset/daemonset(name, namespace)` for rolling restarts (strategic-merge-patch on `kubectl.kubernetes.io/restartedAt` annotation, aiohttp primary + `patch_namespaced_*` fallback).
- **`coordinator.py`** — Extends HA's `DataUpdateCoordinator`. Polls the cluster on an interval (60 s default, 300 s when watch is active), aggregates all resources into lookup dicts, handles orphaned device cleanup. Merges node metrics (`cpu_usage_millicores`, `memory_usage_mib`) from the Metrics API into node data when available. Uses `_data_lock` (`asyncio.Lock`) to prevent watch events from modifying `self.data` during a polling cycle. When watch is enabled, also manages background watch tasks (`async_start_watch_tasks`, `async_stop_watch_tasks`, `_run_watch_loop`, `_apply_watch_event`).
- **`sensor.py`** — Aggregate count sensors (pods, nodes, deployments, cronjobs, jobs, etc.) and per-resource sensors (individual node/pod/cronjob/job metrics).
- **`switch.py`** — Scale control for Deployments/StatefulSets (on=running, off=scaled to 0) and CronJob suspension. `KubernetesReplicaWorkloadSwitch` is the parameterized base class for replica-based workloads; `KubernetesDeploymentSwitch` and `KubernetesStatefulSetSwitch` are thin subclasses that pass workload-specific config (resource key, client methods, log label). `KubernetesCronJobSwitch` remains separate (suspension vs scaling). Includes verification timeouts and cooldowns.
- **`binary_sensor.py`** — Cluster health connectivity indicator and per-node condition binary sensors (MemoryPressure, DiskPressure, PIDPressure, NetworkUnavailable). Supports dynamic discovery of new nodes via coordinator listener (same pattern as `sensor.py`), storing the `async_add_entities` callback and pending unique_ids in `hass.data`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tibuntu/homeassistant-kubernetes](https://github.com/tibuntu/homeassistant-kubernetes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
