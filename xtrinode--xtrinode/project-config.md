---
trigger: always_on
description: This is the single source of repository guidance for humans and coding agents. In this repository,
---

# XTrinode Repository Agent Guide

This is the single source of repository guidance for humans and coding agents. In this repository,
"agent" means a long-running XTrinode runtime process. Reusable Codex skills belong in separate
skill directories with a `SKILL.md`; the project itself should not keep root `SKILLS.md`, `.codex`,
`.agents`, or editor-specific rule directories.

## Repository Guidance Policy

- Keep durable project guidance in this file.
- Do not add package-local `AGENTS.md` files unless a subtree has materially different rules that
  cannot be explained here.
- Do not commit `.codex/`, `.agents/`, or `.cursor/`; those are local/editor state.
- Keep generated, local, and machine-specific files out of git.
- Prefer project conventions over new abstractions. Make focused changes and avoid unrelated
  cleanup in the same patch.

## System Agents

### Operator (`xtrinode/cmd/operator`)

Role: Kubernetes controller manager.

Responsibilities:

- Reconciles `XTrinode` CRs and creates or updates Trino Kubernetes resources.
- Reconciles `XTrinodeCatalog` CRs and catalog ConfigMaps.
- Manages KEDA `ScaledObject` resources for worker autoscaling.
- Drives auto-suspend by patching `spec.suspended = true` when a cluster is idle.
- Drives graceful shutdown by draining running queries before scale-down.
- Runs admission webhooks for `XTrinode` validation.
- Maintains status conditions and emits Kubernetes events.

Interacts with:

- Kubernetes API through controller-runtime.
- KEDA controller through `ScaledObject` CRDs.
- API Server for control-plane operations.

### API Server (`xtrinode/cmd/api-server`)

Role: REST gateway for operator control-plane operations.

Responsibilities:

- Exposes suspend, resume, status, health, and Prometheus metrics endpoints.
- Uses Kubernetes Lease objects to prevent resume stampedes.
- Receives resume and suspend requests from the Gateway and external clients.

Interacts with:

- Kubernetes API for `XTrinode` reads, patches, and Lease management.
- Gateway for upstream resume triggers.

### Gateway (`xtrinode/cmd/gateway`)

Role: HTTP reverse proxy for Trino query routing.

Responsibilities:

- Routes Trino HTTP requests by hostname, `X-Trino-XTrinode`, or default route.
- Maintains sticky query routing so follow-up requests reach the same backend.
- Load-balances across backends in the same routing group.
- Requests resume for suspended or sleeping XTrinodes when no backend is selectable or a coordinator
  connection fails. Plain Trino 503 responses get retry guidance and are not treated as proof of
  suspension.
- Enforces rate limits, circuit breaking, active health checks, and optional API key or bearer-token
  auth.
- Reloads routes dynamically from a ConfigMap.

Interacts with:

- Trino coordinator pods.
- API Server for resume and suspend requests.
- Kubernetes API for route ConfigMaps and auth Secrets.
- Redis when sticky routing or distributed rate limit state is externalized.

## Interaction Overview

```text
External Client / Trino CLI
        |
        v
   [Gateway :8080]
        |-- route lookup --> [ConfigMap: trino-gateway-routes]
        |-- sticky lookup -> [Redis or local LRU cache]
        |-- resume trigger -> [API Server :8081]
        |                          |
        |                          |-- acquire Lease ------> [Kubernetes API]
        |                          `-- patch XTrinode CR --> [Kubernetes API]
        |
        v
  [Trino Coordinator]
        |
        v
  [Trino Workers] <-- ScaledObject -- [KEDA Controller]
                                         ^
                                         |
                                    [Operator]
```

## Configuration Summary

| Agent | Default Port | Namespace | Key Config Source |
| --- | --- | --- | --- |
| Operator | 8081 health | `xtrinode-system` | CLI flags and `internal/config` |
| API Server | 8081 | `xtrinode-system` | CLI flags and `internal/config` |
| Gateway | 8080 | `xtrinode-gateway` | CLI flags and ConfigMap |

## Lifecycle Rules

- Operator leader election uses the `xtrinode-operator-leader-election` Lease.
- All agents handle `SIGINT` and `SIGTERM` with graceful drain timeouts.
- API Server resume operations are serialized with per-runtime or per-routing-group Lease objects.
- Operator auto-suspend checks `spec.autoSuspendAfter` during reconciliation.
- Gateway auto-resume calls the API Server when route state or connection errors indicate a backend
  should be resumed.

## Operator Module Wiring

The operator module hosts three binaries. Entrypoints should stay mostly wiring-only:

| Binary | Startup Pattern |
| --- | --- |
| `cmd/operator/main.go` | Configure scheme, controller-runtime manager, reconcilers, webhooks |
| `cmd/api-server/main.go` | Parse flags, build Kubernetes client, create server, drain on signal |
| `cmd/gateway/main.go` | Parse flags, create auth and route services, start proxy, drain on signal |

Operator dependencies are injected before registering reconcilers:

| Service | Interface | Implementation |
| --- | --- | --- |
| `NodePoolAdapter` | `controllers.NodePoolAdapterInterface` | `controllers.NewNodePoolAdapter` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xtrinode/xtrinode](https://github.com/xtrinode/xtrinode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
