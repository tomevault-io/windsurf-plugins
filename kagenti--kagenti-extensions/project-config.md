---
trigger: always_on
description: This file provides context for Claude (AI assistant) when working with the `kagenti-extensions` monorepo.
---

# CLAUDE.md - Kagenti Extensions

This file provides context for Claude (AI assistant) when working with the `kagenti-extensions` monorepo.

## AI Assistant Instructions

- **Use `Assisted-By` for attribution** — never add `Co-Authored-By`, `Generated with Claude Code`, or similar trailers. See [Commit Attribution Policy](#commit-attribution-policy) below.

## Repository Overview

**kagenti-extensions** contains Kubernetes security extensions for the [Kagenti](https://github.com/kagenti/kagenti) ecosystem. It provides **zero-trust authentication** for Kubernetes workloads through transparent token exchange and dynamic Keycloak client registration using SPIFFE/SPIRE identities.

The sidecar injection webhook lives in a separate repo: [kagenti/kagenti-operator](https://github.com/kagenti/kagenti-operator).

**GitHub:** `github.com/kagenti/kagenti-extensions`
**Container registry:** `ghcr.io/kagenti/kagenti-extensions/<image-name>`
**License:** Apache 2.0

## Top-Level Directory Structure

```
kagenti-extensions/
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
│   ├── cmd/authbridge/       #   Unified binary — 3 modes, 1 codebase
│   │   ├── listener/         #     Protocol adapters (ext_proc, ext_authz, forward/reverse proxy)
│   │   ├── entrypoint.sh     #     Envoy + authbridge process supervision
│   │   └── Dockerfile        #     Combined Envoy + authbridge image
│   ├── authproxy/            #   Auth proxy support files and demos
│   │   ├── quickstart/       #     Standalone demo (no SPIFFE)
│   │   └── k8s/              #     Standalone K8s manifests
│   ├── client-registration/  #   Keycloak auto-registration (Python)
│   ├── spiffe-helper/        #   SPIFFE helper Dockerfile (fetches JWT-SVIDs from SPIRE)
│   ├── demos/                #   Demo scenarios (weather-agent, github-issue, webhook, single-target, multi-target)
│   └── keycloak_sync.py      #   Declarative Keycloak sync tool
├── tests/                    # Python tests (client-registration, keycloak_sync)
├── .github/
│   ├── workflows/            # CI/CD (ci.yaml, build.yaml, security-scans, scorecard, spellcheck)
│   └── ISSUE_TEMPLATE/       # Bug report, feature request, epic templates
├── .pre-commit-config.yaml   # Pre-commit hooks (trailing whitespace, go fmt/vet, ruff)
└── CLAUDE.md                 # This file
```

## Major Components

### 1. AuthBridge Unified Binary (Go)

A **single binary** providing transparent traffic interception for both inbound JWT validation and outbound OAuth 2.0 token exchange (RFC 8693), supporting three deployment modes.

**Location:** `authbridge/cmd/authbridge/`
**Library:** `authbridge/authlib/`
**Language:** Go 1.24
**Detailed guide:** [`authbridge/CLAUDE.md`](authbridge/CLAUDE.md)

**Core components:**
- `cmd/authbridge/main.go` — Unified binary (ext_proc, ext_authz, forward/reverse proxy modes)
- `authlib/` — Shared auth library (JWT validation, token exchange, caching, routing)
- `authproxy/init-iptables.sh` — Traffic interception setup (Istio ambient mesh compatible)
- `authproxy/Dockerfile.init` — Init container image

**Ports:** 15123 (outbound), 15124 (inbound), 9090 (ext-proc/ext-authz), 9901 (admin), 9093 (stats and config)

### 2. Client Registration (Python)

A Python script that **automatically registers Kubernetes workloads as Keycloak OAuth2 clients** using their SPIFFE identity.

**Location:** `authbridge/client-registration/`
**Language:** Python 3.12
**Detailed guide:** [`authbridge/CLAUDE.md`](authbridge/CLAUDE.md)

**Flow:** Reads SPIFFE ID from JWT, registers client in Keycloak, writes secret to `/shared/client-secret.txt`

## How the Components Work Together

The kagenti-operator (in a separate repo) injects AuthBridge sidecars into workload pods. Once injected, the sidecars work together:

```
         ┌────────────────────────────────────┐
         │            WORKLOAD POD            │
         │                                    │
         │  proxy-init (init) ─► iptables     │
         │                                    │
         │  spiffe-helper ──► SPIRE Agent     │
         │       │ writes JWT SVID            │
         │       ▼                            │
         │  client-registration ──► Keycloak  │
         │       │ writes client secret       │
         │       ▼                            │
         │  envoy-proxy (+ authbridge)        │
         │    - Inbound: JWT validation       │
         │    - Outbound: token exchange       │
         │       │                            │
         │  Your Application                  │
         └────────────────────────────────────┘
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kagenti/kagenti-extensions](https://github.com/kagenti/kagenti-extensions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
