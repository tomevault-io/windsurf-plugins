---
trigger: always_on
description: This file provides context for Claude (AI assistant) when working with the `cortex` monorepo.
---

# CLAUDE.md - Rossoctl Extensions

This file provides context for Claude (AI assistant) when working with the `cortex` monorepo.

## AI Assistant Instructions

- **Use `Assisted-By` for attribution** — never add `Co-Authored-By`, `Generated with Claude Code`, or similar trailers. See [Commit Attribution Policy](#commit-attribution-policy) below.

## Repository Overview

**cortex** contains Kubernetes security extensions for the [Rossoctl](https://github.com/rossoctl/rossoctl) ecosystem. It provides **zero-trust authentication** for Kubernetes workloads through transparent token exchange and dynamic Keycloak client registration using SPIFFE/SPIRE identities.

The sidecar injection webhook lives in a separate repo: [rossoctl/operator](https://github.com/rossoctl/operator).

**GitHub:** `github.com/rossoctl/cortex`
**Container registry:** `ghcr.io/rossoctl/cortex/<image-name>`
**License:** Apache 2.0

## Top-Level Directory Structure

```
cortex/
├── authbridge/               # Authentication bridge components
│   ├── authlib/              #   Shared auth building blocks (Go module)
│   │   ├── validation/       #     JWKS-backed JWT verifier
│   │   ├── exchange/         #     RFC 8693 token exchange client
│   │   ├── cache/            #     SHA-256 keyed token cache
│   │   ├── bypass/           #     Path pattern matcher
│   │   ├── spiffe/           #     SPIFFE credential sources
│   │   ├── routing/          #     Host-to-audience router
│   │   ├── auth/             #     HandleInbound + HandleOutbound composition
│   │   └── config/           #     Mode presets, YAML config, validation
│   ├── cmd/authbridge-proxy/ #   proxy-sidecar mode (default): HTTP forward + reverse
│   │   │                     #   proxies, full plugin set including parsers
│   │   ├── main.go
│   │   ├── Dockerfile        #     proxy-sidecar combined image
│   │   └── entrypoint.sh
│   ├── cmd/authbridge-envoy/ #   envoy-sidecar mode: ext_proc gRPC server, full plugin set
│   │   ├── main.go
│   │   ├── Dockerfile        #     envoy-sidecar combined image
│   │   └── entrypoint.sh
│   │                        #   (the authbridge-lite image is this proxy
│   │                        #    binary built with exclude_plugin_* tags)
│   ├── proxy-init/           #   iptables init container (envoy-sidecar + proxy-sidecar enforce-redirect modes)
│   │   ├── init-iptables.sh
│   │   ├── Dockerfile.init
│   │   ├── Makefile
│   │   └── README.md
│   ├── demos/                #   Demo scenarios (weather-agent, github-issue, token-exchange-routes, mcp-parser)
│   └── keycloak_sync.py      #   Declarative Keycloak sync tool
├── tests/                    # Python tests (keycloak_sync)
├── .github/
│   ├── workflows/            # CI/CD (ci.yaml, build.yaml, security-scans, scorecard, spellcheck)
│   └── ISSUE_TEMPLATE/       # Bug report, feature request, epic templates
├── .pre-commit-config.yaml   # Pre-commit hooks (trailing whitespace, go fmt/vet, ruff)
└── CLAUDE.md                 # This file
```

## Major Components

### 1. AuthBridge Binaries (Go)

**Two mode-specific binaries** (proxy-sidecar and envoy-sidecar) providing transparent traffic interception for both inbound JWT validation and outbound OAuth 2.0 token exchange (RFC 8693). Each binary is hardcoded to its deployment shape; mode is no longer selected at runtime. The `authbridge-lite` **image** is a build variant of the proxy binary (not a third binary) — see below.

**Library:** `authbridge/authlib/` (shared)
**Language:** Go 1.25
**Detailed guide:** [`authbridge/CLAUDE.md`](authbridge/CLAUDE.md)

**Binaries:**
- `cmd/authbridge-proxy/` — proxy-sidecar mode (default): HTTP forward + reverse proxies, full plugin set (jwt-validation, token-exchange, a2a-parser, mcp-parser, inference-parser). No Envoy / no gRPC.
- `cmd/authbridge-envoy/` — envoy-sidecar mode: ext_proc gRPC server hooked into Envoy, full plugin set.
- `authbridge-lite` (image, **not** a separate binary) — `cmd/authbridge-proxy` built with `exclude_plugin_*` tags so only jwt-validation + token-exchange compile in (OPA + parsers dropped), for size-optimized deployments. Individual plugins live in `cmd/authbridge-proxy/plugins_<name>.go` files gated by `//go:build !exclude_plugin_<name>`.

**Common:**
- `authlib/` — shared auth library (JWT validation, token exchange, caching, routing, all listener implementations, all plugins).
- `proxy-init/init-iptables.sh` — traffic interception setup (Istio ambient mesh compatible). Used by envoy-sidecar mode (`redirect`) and by proxy-sidecar mode's `enforce-redirect` egress guard.
- `proxy-init/Dockerfile.init` — proxy-init container image.

**Ports (envoy-sidecar):** 15123 (outbound), 15124 (inbound), 9090 (ext-proc), 9901 (admin)
**Ports (proxy-sidecar / lite):** 8080 (reverse proxy), 8081 (forward proxy), 9091 (health), 9093 (stats), 9094 (session API)

### 2. Client Registration

Keycloak client registration for workloads is handled by the
**operator** (separate repo) — see `operator/docs/operator-managed-client-registration.md`.
The operator creates a Secret with `client-id.txt` + `client-secret.txt`
and the webhook mounts it at `/shared/` in the workload pod. The
in-pod `client-registration` sidecar that previously lived in this

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rossoctl/cortex](https://github.com/rossoctl/cortex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
