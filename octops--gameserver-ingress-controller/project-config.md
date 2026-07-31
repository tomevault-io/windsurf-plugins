---
trigger: always_on
description: Guidance for AI agents working in this repository.
---

# AGENTS.md — Gameserver Ingress Controller

Guidance for AI agents working in this repository.

---

## What This Project Does

This is a **Kubernetes controller** that automatically provisions network ingress resources for game servers managed by [Agones](https://agones.dev/). When Agones creates a `GameServer`, this controller watches for it and creates the corresponding `Service`, `Ingress` (or `HTTPRoute` for Gateway API), and annotates the `GameServer` as ingress-ready.

**Without this controller**, game server operators would manually manage hundreds of ingress rules as game servers spin up/down dynamically. This automates the full lifecycle.

**Two routing backends are supported:**
- `ingress` (default) — standard `networking.k8s.io/v1` Ingress, one per game server, TLS via cert-manager per server
- `gateway` — Kubernetes Gateway API (`gateway.networking.k8s.io/v1`) HTTPRoute, attaches to a shared pre-provisioned Gateway; no per-server TLS (cert on the Gateway listener instead)

**Two routing modes** apply to both backends:
- `domain` — each server gets a subdomain: `<gs-name>.<domain>`
- `path` — all servers share an FQDN with path-based routing: `<fqdn>/<gs-name>`

---

## Repository Layout

```
cmd/                    CLI entry point (Cobra)
pkg/
  app/                  Bootstrap: wires manager, stores, handler, controller
  controller/           controller-runtime Reconciler that watches GameServer objects
  handlers/             EventHandler: OnAdd/OnUpdate/OnDelete → calls reconcilers
  reconcilers/          Business logic per resource type
    service_reconciler.go
    ingress_reconciler.go
    gateway_reconciler.go   ← new Gateway API feature
    gameserver_reconciler.go
    *_options.go            ← functional options for each reconciler
  stores/               Kubernetes API abstraction (informer-backed caches)
    store.go            ← composite store wiring
    gateway_store.go    ← new Gateway API store
  gameserver/           Constants, annotation helpers, GameServer utility funcs
  record/               Kubernetes event recorder wrapper
  k8sutil/              Cluster config and client helpers
  manager/              controller-runtime manager setup
internal/
  runtime/              Logger (logrus) and signal handling
  version/              Build metadata
deploy/
  install.yaml          Controller RBAC + Deployment manifest
  gateway/              Gateway API infrastructure (GatewayClass for Contour provisioner)
  cert-manager/         ClusterIssuer examples — ingress mode only
examples/               Sample Fleet manifests for all routing modes and backends
docs/                   Design documents (gateway-api-support-plan.md)
```

---

## Core Data Flow

```
GameServer created/updated in Kubernetes
        │
        ▼
GameServerController (controller-runtime Reconciler)
        │
        ▼
GameSeverEventHandler.OnAdd / OnUpdate
        │  skip if: missing octops.io/gameserver-ingress-mode annotation
        │  skip if: GameServer in Shutdown state
        │  only proceed for: Scheduled, RequestReady, Ready states
        │
        ▼
Reconcile() — sequential pipeline:
  1. ServiceReconciler.Reconcile()        → headless ClusterIP Service
  2. IngressReconciler OR GatewayReconciler.Reconcile()
       IngressReconciler  → networking.k8s.io/v1 Ingress
       GatewayReconciler  → gateway.networking.k8s.io/v1 HTTPRoute
  3. GameServerReconciler.Reconcile()     → patch octops.io/ingress-ready=true
        │
        ▼
Kubernetes Events recorded for audit (via record.EventRecorder)
```

All created resources carry an **OwnerReference** back to the `GameServer`, so deleting the `GameServer` cascades to cleanup automatically.

---

## Key Annotations Reference

All annotations live under the `octops.io/` prefix and are set on `GameServer` / `Fleet` spec templates.

| Annotation | Required | Values | Notes |
|---|---|---|---|
| `octops.io/gameserver-ingress-mode` | yes | `domain` \| `path` | Missing = skip reconciliation entirely |
| `octops.io/router-backend` | no | `ingress` (default) \| `gateway` | Selects backend |
| `octops.io/gameserver-ingress-domain` | domain mode | e.g. `game.example.com` | Used as base domain |
| `octops.io/gameserver-ingress-fqdn` | path mode | e.g. `servers.example.com` | Used as hostname |
| `octops.io/terminate-tls` | no | `true` \| `false` | Ingress only; warns if set in gateway mode |
| `octops.io/tls-secret-name` | no | secret name | Custom TLS secret |
| `octops.io/issuer-tls-name` | no | ClusterIssuer name | Adds `cert-manager.io/cluster-issuer`; warns if set in gateway mode |
| `octops.io/ingress-class-name` | no | e.g. `contour` | Sets `spec.ingressClassName` |
| `octops.io/ingress-ready` | internal | `true` | Written by controller, do not set manually |
| `octops.io/gateway-name` | gateway mode | Gateway resource name | Required when `router-backend=gateway` |
| `octops.io/gateway-namespace` | no | namespace | Defaults to GameServer namespace |
| `octops.io/gateway-section-name` | no | listener name | Optional Gateway listener |

**Custom annotation forwarding:**
- `octops-<key>: <value>` → forwarded to Ingress/HTTPRoute as `<key>: <value>` (prefix stripped)
- `octops.service-<key>: <value>` → forwarded to Service as `<key>: <value>` (prefix stripped)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Octops/gameserver-ingress-controller](https://github.com/Octops/gameserver-ingress-controller) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
