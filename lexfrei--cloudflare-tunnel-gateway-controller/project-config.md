---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Kubernetes controller implementing Gateway API for Cloudflare Tunnel. Watches Gateway and HTTPRoute resources, automatically configures Cloudflare Tunnel ingress rules via API. Supports hot reload without cloudflared restart. Optional AmneziaWG (AWG) sidecar support for traffic obfuscation.

## Build and Development Commands

```bash
# Build binary
go build -o bin/controller ./cmd/controller

# Build with version info
go build -ldflags "-X main.Version=v0.0.1 -X main.Gitsha=$(git rev-parse HEAD)" -o bin/controller ./cmd/controller

# Run tests
go test -v -race -coverprofile=coverage.out ./...

# Run single test
go test -v -race ./internal/dns/... -run TestDetectClusterDomain

# Run linter (all errors must be fixed before committing)
golangci-lint run --timeout=5m

# Lint markdown files
markdownlint-cli2 '**/*.md'

# Build proxy binary
go build -o bin/proxy ./cmd/proxy

# Build container
podman build --tag cloudflare-tunnel-gateway-controller:dev --file Containerfile .
```

## Helm Chart Commands

```bash
# Package chart
helm package charts/cloudflare-tunnel-gateway-controller

# Run helm-unittest
helm unittest charts/cloudflare-tunnel-gateway-controller

# Generate README from values.yaml (REQUIRED before commit)
helm-docs charts/cloudflare-tunnel-gateway-controller

# Lint chart
helm lint charts/cloudflare-tunnel-gateway-controller

# Template locally (for debugging)
helm template test charts/cloudflare-tunnel-gateway-controller --values charts/cloudflare-tunnel-gateway-controller/examples/basic-values.yaml
```

## Architecture

### Controllers (controller-runtime based)

- **GatewayReconciler** (`internal/controller/gateway_controller.go`): Watches Gateway resources whose GatewayClass has a matching `spec.controllerName`. Resolves GatewayClassConfig for tunnel credentials. Optionally manages cloudflared deployment via Helm SDK. Updates Gateway status with tunnel CNAME address.

- **HTTPRouteReconciler** (`internal/controller/httproute_controller.go`): Watches HTTPRoute resources referencing managed Gateways. Performs full sync of all relevant routes to Cloudflare Tunnel configuration on any change. Updates HTTPRoute status. Optionally pushes config to v2 proxy replicas via ProxySyncer.

- **GRPCRouteReconciler** (`internal/controller/grpcroute_controller.go`): Watches GRPCRoute resources. Shares RouteSyncer with HTTPRouteReconciler for unified Cloudflare Tunnel sync. Does NOT push to v2 proxy (GRPC is tunnel-only).

- **ProxySyncer** (`internal/controller/proxy_syncer.go`): Converts HTTPRoutes into proxy config and pushes to proxy endpoints via HTTP API. Resolves headless service DNS for endpoint discovery. Validates cross-namespace backends via ReferenceGrant.

### Custom Resource Definition

- **GatewayClassConfig** (`api/v1alpha1/`): Cluster-scoped CRD for configuring tunnel credentials and cloudflared deployment. Referenced by GatewayClass via `parametersRef`. Supports AWG sidecar configuration.

### Supporting Packages

- **internal/config/resolver.go**: Resolves GatewayClassConfig from GatewayClass parametersRef, reads credentials from Secrets, auto-detects account ID via Cloudflare API.

- **internal/ingress/builder.go**: Converts HTTPRoute specs to Cloudflare tunnel ingress rules. Handles hostnames, path matching (prefix/exact), backend service resolution.

- **internal/helm/manager.go**: Helm SDK wrapper for installing/upgrading cloudflared from OCI registry (`oci://ghcr.io/lexfrei/charts/cloudflare-tunnel`). Includes chart version caching.

- **internal/dns/detect.go**: Auto-detects Kubernetes cluster domain from `/etc/resolv.conf` search domains.

### V2 Proxy Data Plane

- **cmd/proxy/**: Standalone binary running cloudflared tunnel transport with an L7 reverse proxy. Supports tunnel mode (in-process via `OriginProxy`) and standalone mode (HTTP server for development).

- **internal/proxy/**: L7 reverse proxy implementation:
  - `router.go` — Thread-safe HTTP router with atomic config swap, compiled matchers, and weighted backend selection.
  - `handler.go` — Request handler with filter pipeline, per-route timeouts, and transport pool management.
  - `converter.go` — Converts Gateway API HTTPRoute resources to proxy config.
  - `filter.go` — Filter implementations: header modifier, redirect, URL rewrite, request mirror.
  - `config.go` — Proxy config types (rules, matches, filters, backends, timeouts).
  - `api.go` — Config API endpoints (PUT/GET /config, healthz, readyz) with Bearer token auth.
  - `pusher.go` — HTTP client for pushing config to proxy replicas concurrently.

- **internal/tunnel/**: Bootstraps cloudflared tunnel with in-process proxy override. Parses tunnel tokens, builds tunnel configuration, and integrates with the vendored cloudflared `OverrideProxy` hook. Exposes `GatewayOriginProxy` adapter for routing requests to the L7 proxy.

### Key Dependencies

- `sigs.k8s.io/controller-runtime` - Kubernetes controller framework
- `sigs.k8s.io/gateway-api` - Gateway API types
- `github.com/cloudflare/cloudflare-go/v6` - Cloudflare API client

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lexfrei/cloudflare-tunnel-gateway-controller](https://github.com/lexfrei/cloudflare-tunnel-gateway-controller) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
