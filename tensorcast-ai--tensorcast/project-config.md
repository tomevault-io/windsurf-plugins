---
trigger: always_on
description: This file provides guidance to AI when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI when working with code in this repository.

## Project Overview

TensorCast is a high-performance distributed artifact storage and loading system. It uses a distributed master-worker architecture; see Architecture Overview below for details.

## Artifact-First Architecture Principle (Required)

TensorCast should keep artifact as the primary system abstraction for durable
identity, data movement, discovery, routing, and lifecycle. When a new workflow
needs a capability that looks adjacent to loading, serving, publication, or
sharing, first ask how the artifact model should express it and extend that
model deeply.

Avoid introducing parallel concepts, side channels, or feature-specific source
authorities when the capability can be represented as stronger artifact
metadata, artifact lifecycle, artifact publication, or artifact replica
semantics. Add a new concept only when the artifact model is clearly
insufficient, and document why extending artifact would be the wrong abstraction.

## Command Execution

- You must use `pytest tests/python/xxxx` to run python tests
- You must use `bazel test //core/component:xxx_test` to run cxx tests
- These policies keep virtualenv isolation consistent; violating them can break the build and introduce environment skew.

## Python SDK Daemon Startup (Quick Map)

- Store API calls (e.g., `tc.from_disk`) go through `tensorcast.api.store.runtime.get_context()`; if no runtime exists it tries `startup.init(mode="connect")` and falls back to `startup.init(mode="create")`.
- `startup.init(mode="connect")` resolves the daemon address from an explicit parameter or the current local session (`runtime.status()` / service manager); it errors if no session is found or the daemon is unreachable.
- `startup.init(mode="create")` launches a local daemon via `runtime.start(...)` using a daemon config path selected in order: explicit `daemon_config_path` → `$TENSORCAST_DAEMON_CONFIG` → `examples/config/store_daemon_config.yaml` (repo or packaged wheel).
- If the chosen config enables HA or disk operations, the daemon must have `daemon_id` set (for Global Store registration) and `server.storage_path` set (for disk materialization). Missing values cause startup to exit early with clear errors.
- **Hard rule (required)**: Python SDK code MUST NOT connect to Global Store directly (no direct gRPC channel/stub, no Global Store endpoint usage). SDK key-mapping, artifact metadata, and control-path operations MUST go through the Store Daemon APIs only.

### Local GS/Daemon Startup (Blocking, Repro Baseline)

Use the following startup order for local debugging/repro:

```bash
# Terminal 1: start Global Store (blocking)
source .venv/bin/activate
tensorcast-cli global start --blocking

# Terminal 2: start Store Daemon (blocking), connect to local GS
source .venv/bin/activate
LD_LIBRARY_PATH=/data/cuda/compat tensorcast-cli daemon start --blocking \
  --global-store-mode connect \
  --global-store-address 127.0.0.1:50051
```

Optional cleanup commands:

```bash
source .venv/bin/activate
tensorcast-cli daemon stop
tensorcast-cli global stop
```

### Multi-host Daemon Orchestration (Required)

For cross-host tests (including WeightPublisher and fanout benchmarks), use the
following daemon orchestration rules:

- Assign a unique local daemon listen/connect port per node (or per role process)
  and keep it stable for the whole case. Do not reuse a single default port
  across all roles.
- Manage daemon lifecycle explicitly in the runner:
  - Pre-clean stale daemon sessions/processes before starting a case.
  - Start daemon with an explicit session id and verify ready status before
    launching role workloads.
  - Stop the same explicit session during teardown (including failure paths).
- Run Global Store on the local control host by default for cross-host cases.
  Do not colocate GS with role workers unless the case explicitly requires it.
  This avoids worker-scoped daemon lifecycle cleanup from stopping GS.
- Do not rely on implicit SDK auto-create startup in multi-host tests; role
  scripts should connect to a known local daemon endpoint.
- Keep SDK connectivity local-only on each node: app processes connect to their
  own node-local daemon address, never directly to a remote daemon or GS.

## Architecture Overview

### Runtime Topology
```
                          ┌─────────────────────────────┐
                          │         Global Store        │
                          │  (Metadata & Coordination)  │
                          └──────────────┬──────────────┘
                                         │ gRPC (metadata only)
                ┌────────────────────────┴────────────────────────┐
                │                                               │
    ┌───────────▼───────────┐                       ┌───────────▼───────────┐
    │   Store Daemon #1     │  RDMA/TCP (P2P data)  │    Store Daemon #N    │
    │  (C++ over StoreEngine)│<-------------------->│  (C++ over StoreEngine)│
    │ - VS/UMA memory       │                       │ - VS/UMA memory       │
    │ - Disk & P2P loaders  │                       │ - Disk & P2P loaders  │
    │ - CUDA IPC export     │                       │ - CUDA IPC export     │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tensorcast-ai/tensorcast](https://github.com/tensorcast-ai/tensorcast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-26 -->
