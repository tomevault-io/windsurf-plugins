---
trigger: always_on
description: Zero-trust access gateway bridging Twingate with L7 resources such as Kubernetes API servers, HTTP services, and SSH servers. Authentication uses JWT + Proof-of-Possession.
---

# Twingate Gateway - AI Development Guide

## Project Overview

Zero-trust access gateway bridging Twingate with L7 resources such as Kubernetes API servers, HTTP services, and SSH servers. Authentication uses JWT + Proof-of-Possession.

- **License**: MPL-2.0
- **Repository**: <https://github.com/Twingate/gateway>
- **Language**: Go 1.26.2
- **Build**: goreleaser, Docker buildx, kind (testing)
- **Linting**: golangci-lint v2.11.1
- **Testing**: testify, helm-unittest

**Key Features**: TLS 1.3 mutual auth, K8s user impersonation, SSH certificate-based access, session recording, Prometheus metrics

**Core Dependencies**: k8s.io/client-go, golang.org/x/crypto/ssh, github.com/golang-jwt/jwt, go.uber.org/zap, prometheus/client_golang

## Architecture

### Startup Flow

```text
main.go → cmd/start.go → proxy.NewProxy() → proxy.Start()
  ├─> connect.NewListener() (TLS + protocol multiplexing)
  ├─> httphandler.NewProxy().Start() (K8s API proxy)
  ├─> sshhandler.NewProxy().Start() (SSH proxy)
  └─> metrics.Start() (Prometheus)
```

### Core Components

**`internal/proxy/proxy.go`** - Central orchestrator using errgroup for lifecycle management

**`internal/connect/`** - Connection handling:

- `listener.go`: Protocol multiplexer (HTTP/SSH based on handshake)
- `connect.go`: Validates CONNECT + JWT + Proof-of-Possession (EKM signature)
- `cert_reloader.go`: Hot-reloads TLS certs without restart

**`internal/httphandler/`** - Kubernetes proxy:

- Reverse proxy to K8s API with user impersonation headers
- Multiple upstreams (in-cluster or external with custom tokens)
- WebSocket support for kubectl exec/logs

**`internal/sshhandler/`** - SSH proxy:

- SSH server with CA-signed certificates (auto/manual/Vault)
- Bidirectional channel forwarding to upstreams
- Host certs (gateway→client) + User certs (gateway→upstream)

**`internal/token/`** - JWT validation:

- Fetches JWKS from Twingate
- Validates GAT claims (user, resource, client public key)
- Proof-of-Possession: client signs TLS EKM with private key

### Security Model

**Zero-Trust**: No stored credentials, per-connection JWT validation, TLS 1.3 mutual auth

**Token Validation Flow**:

1. Client sends CONNECT with `Proxy-Authorization: Bearer <JWT>` + `X-Token-Signature: <ECDSA_sig(TLS_EKM)>`
2. Gateway validates JWT signature via JWKS
3. Gateway verifies PoP signature using client public key from JWT
4. Gateway checks requested address matches token resource
5. Connection allowed, user identity extracted

**K8s Security**: Gateway uses impersonation headers (`Impersonate-User`, `Impersonate-Group`). K8s RBAC enforced at API server level. Gateway service account only needs impersonation permission.

**SSH Security**: Certificate-based auth. CA options: auto-generated (testing), manual (file), Vault (production). Separate CAs supported for gateway-host, gateway-user, and upstream verification.

## Directory Structure

```text
.
├── cmd/                    # CLI (Cobra)
├── internal/
│   ├── config/             # YAML config + validation
│   ├── connect/            # TLS + protocol multiplexing
│   ├── httphandler/        # K8s API proxy
│   ├── sshhandler/         # SSH proxy
│   ├── token/              # JWT validation
│   ├── sessionrecorder/    # Audit logs
│   ├── metrics/            # Prometheus
│   ├── log/                # Logging
│   └── version/
├── deploy/gateway/         # Helm chart
├── test/
│   ├── integration/        # Kind cluster tests
│   ├── e2e/                # End-to-end tests
│   └── data/               # Test fixtures
├── .github/workflows/      # CI/CD
├── main.go
├── Makefile
├── .goreleaser.yaml
├── .golangci.yml
└── go.mod
```

## Development Workflows

### Setup

```bash
asdf install golang 1.26.2
```

Versions tracked in `.tool-versions`.

### Commands

| Task | Command |
|------|---------|
| Build | `make build` |
| Test (Unit) | `make test` |
| Test (Integration) | `make test-integration` |
| Test (E2E) | `make test-e2e` |
| Test (Helm) | `make test-helm` |
| Lint Go | `make lint` |
| Lint Dockerfile | `make lint-dockerfile` |
| Lint Markdown | `make lint-markdown` |
| Version | `make version` |
| Cut Dev Release | `make cut-release-dev` |
| Cut Prod Release | `make cut-release-prod` |

### Before Committing

**IMPORTANT**: Run appropriate checks:

- **Go code** (`.go`): `make lint && make test`
- **Dockerfiles**: `make lint-dockerfile`
- **Markdown** (`*.md`): `make lint-markdown`
- **Helm charts**: `make test-helm`

**Full suite**: `make lint && make lint-dockerfile && make lint-markdown && make test && make test-integration && make test-helm`

### Testing

- **Unit**: `./cmd/...`, `./internal/...` - table-driven tests with testify
- **Integration**: `./test/integration/...` - requires kind cluster, tests full flows
- **E2E**: `./test/e2e/...` - full deployment scenario
  - **Prerequisite**: Caddy must be running (`caddy run` or `caddy start`)
  - Acts as reverse proxy for `acme.test` domain to simulate Twingate controller
- **Helm**: `deploy/gateway/tests/...` - snapshot tests

### Release Process

From `master` branch only:

- Dev: `make cut-release-dev` → `v1.2.3-dev-abc1234`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Twingate/gateway](https://github.com/Twingate/gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
