---
trigger: always_on
description: Kubernetes Gateway API implementation using Varnish. Three components:
---

# Varnish Gateway Operator - Development Guide

## Project Overview

Kubernetes Gateway API implementation using Varnish. Three components:
- **operator**: watches Gateway API resources, generates ghost.json config, manages deployments
- **chaperone**: handles endpoint discovery and triggers ghost reload
- **ghost**: Rust VMOD that handles all routing logic internally

## Documentation

- [Documentation Index](docs/index.md) - user-facing docs (getting started, concepts, guides, operations, reference)
- [GatewayClassParameters Reference](docs/reference/gatewayclassparameters.md) - GatewayClassParameters, varnishd args, defaults

## Current Status

The project passes the Gateway API conformance test suite (`make test-conformance`). Remaining work is tracked in GitHub issues.

### Component Overview

**Operator** (`cmd/operator/main.go`):
- Gateway controller - creates Deployment, Service, ConfigMap, Secret, ServiceAccount
- HTTPRoute controller - watches routes, regenerates routing.json on changes
- Status conditions (Accepted/Programmed) on Gateway and HTTPRoute
- GatewayClassParameters CRD for user VCL injection and varnishd extra args
- Client-side TLS termination with cert-manager support and hot-reload

**Chaperone** (`cmd/chaperone/main.go`):
- Starts and manages varnishd via vrun package
- Watches routing.json + EndpointSlices, merges into ghost.json
- Triggers ghost reload via HTTP, VCL reload via varnishadm
- TLS cert loading and hot-reload via fsnotify

**Ghost VMOD** (`ghost/`):
- Rust-based VMOD handling all routing inside Varnish
- Path matching (exact, prefix, regex), method, header, query parameter matching
- Priority-based route selection with additive specificity bonuses
- Hot-reload via `/.varnish-ghost/reload`
- See `ghost/CLAUDE.md` and `ghost/README.md` for details

### Key Packages

- `internal/ghost/` - Config types, ghost.json generator, EndpointSlice watcher
- `internal/vcl/` - VCL generator, merge, hot-reload via varnishadm
- `internal/varnishadm/` - Full varnishadm protocol (reverse mode, -M flag)
- `internal/vrun/` - varnishd process lifecycle management
- `internal/reload/` - HTTP client for ghost reload endpoint

## Development Setup

### Prerequisites

- Go 1.21+
- Rancher Desktop (or any local k8s)
- kubectl configured to talk to your cluster
- Rust 1.75+ (for ghost VMOD development)

### Install Gateway API CRDs

```bash
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.5.0/standard-install.yaml
```

### Local k8s development setup

See k8s-dev-howto.md for instruction on how to setup a local k8s cluster and how to induce noise and chaos into
the cluster.

### Version Management

The `.version` file and git tags are managed by [bump](https://github.com/perbu/bump). It's installed as a Go tool dependency.

```bash
# Bump patch version (v0.1.2 -> v0.1.3)
go tool bump -patch

# Bump minor version (v0.1.2 -> v0.2.0)
go tool bump -minor

# Bump major version (v0.1.2 -> v1.0.0)
go tool bump -major
```

## Architecture

### Data Flow

1. **Operator** watches Gateway/HTTPRoute resources
   - Generates `routing.json` (host → service mappings) and stores in ConfigMap
   - Generates ghost preamble VCL and merges with user VCL
   - Creates Varnish listeners from Gateway listeners (one per unique port)

2. **Chaperone** runs as wrapper for varnishd
   - Starts varnishd via vrun package with named listeners from `VARNISH_LISTEN`
   - Watches `routing.json` + EndpointSlices
   - Merges them into `ghost.json` (host → actual pod IPs)
   - Triggers ghost reload via HTTP endpoint

3. **Ghost VMOD** handles all routing inside Varnish
   - Reads `ghost.json` on reload
   - Routes requests by hostname to weighted backend groups
   - Filters routes by `local.socket` for listener-aware routing
   - Sets `X-Gateway-Listener` header on requests for user VCL

### Multi-Listener Architecture

Each Gateway listener maps to a Varnish `-a` socket named `{proto}-{port}` (e.g., `http-80`, `https-443`).
Multiple Gateway listeners on the same port collapse into a single Varnish socket — hostname
isolation is handled by ghost's existing vhost routing.

**Container ports = listener ports**: No port translation. Listener port 3000 means Varnish
binds to `:3000` inside the container. Listener changes trigger a pod restart (via infra hash).

**Listener naming convention**: `{proto}-{port}` encodes both protocol and port:
- `http-80` — HTTP on port 80
- `https-443` — HTTPS on port 443
- `http-3000` — HTTP on port 3000

Ghost uses `local.socket()` to get the Varnish socket name and filters routes accordingly.
The `listeners` field in routing.json/ghost.json contains socket names that a route applies to.
An empty `listeners` list means the route matches on all listeners.

**Request headers**: Ghost sets two headers on every request for use in user VCL:
- `X-Gateway-Listener` — Varnish socket name (e.g., `http-80`)
- `X-Gateway-Route` — HTTPRoute namespace/name (e.g., `default/my-route`)

These propagate to the backend and are available in user VCL:

```vcl
sub vcl_recv {
    if (req.http.X-Gateway-Listener == "http-80") {
        // redirect to HTTPS, restrict access, etc.
    }
    if (req.http.X-Gateway-Route == "production/api-route") {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [varnish/gateway](https://github.com/varnish/gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
