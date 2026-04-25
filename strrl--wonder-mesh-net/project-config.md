---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Wonder Mesh Net is a PaaS bootstrapper that turns homelab/edge machines (behind NAT, dynamic IPs, firewalls) into bring-your-own compute for PaaS platforms and orchestration tools.

**Core Purpose**: Make scattered compute usable by PaaS/cluster orchestrators as if it were cloud VMs.

**We solve bootstrapping for BYO compute: identity + join tokens + secure connectivity.** App management is left to Kubernetes, PaaS platforms (Zeabur, Railway, Fly.io), or self-hosted PaaS (Coolify, Dokploy).

**Technology**: Tailscale/Headscale for WireGuard-based mesh networking with DERP relay fallback.

## Build Commands

```bash
make build          # Build wonder binary with web UI to bin/
make build-go       # Build Go binary only (skip web UI build)
make build-all      # Cross-compile for linux/darwin, amd64/arm64
make test           # Run tests with race detector
make check          # Run gofmt, go vet, golangci-lint
make generate       # Regenerate sqlc code after schema changes
make clean          # Remove build artifacts
```

Run a single test:
```bash
go test -v -run TestFunctionName ./path/to/package
```

Build artifacts go to `bin/` (gitignored).

## Docker Image

Build and push multi-arch image (linux/amd64 + linux/arm64):

```bash
IMAGE_TAG=v2025.12.07.rev1 ./hack/build-image.sh
```

The script uses `docker buildx` to build for both architectures and pushes to `ghcr.io/strrl/wonder-mesh-net`.

## Web UI

The web UI is a React/TypeScript SPA in `webui/`:

```bash
cd webui && npm ci && npm run build    # Build UI (copied to internal/app/coordinator/webui/static/)
cd webui && npm run dev                # Development server with hot reload
```

## Helm Chart

The Helm chart is in `charts/wonder-mesh-net/`. It deploys coordinator + Headscale (sidecar) + optional Keycloak/PostgreSQL.

### Local Development

```bash
helm lint charts/wonder-mesh-net                    # Lint chart
helm template charts/wonder-mesh-net                # Render templates locally
helm install wonder ./charts/wonder-mesh-net -n wonder --create-namespace
```

### Key Values

| Parameter | Description | Default |
|-----------|-------------|---------|
| `headscale.enabled` | Enable embedded Headscale sidecar | `true` |
| `keycloak.enabled` | Enable embedded Keycloak (dev/test) | `true` |
| `keycloak.production` | Keycloak production mode (uses PostgreSQL) | `false` |
| `postgres.enabled` | Enable PostgreSQL for Keycloak | `false` |
| `coordinator.publicUrl` | Public URL (must match Ingress) | `http://localhost:9080` |
| `coordinator.database.driver` | Database driver (`sqlite` or `postgres`) | `sqlite` |
| `coordinator.jwtSecret` | JWT signing secret (auto-generated if empty) | `""` |

### E2E Testing

The Helm E2E test (`charts/test/test.sh`) runs on Minikube and tests both SQLite and PostgreSQL backends:

```bash
./charts/test/test.sh sqlite    # Test with SQLite backend
./charts/test/test.sh pgsql     # Test with PostgreSQL backend
```

The test:
1. Builds coordinator image and loads into Minikube
2. Installs Helm chart with Keycloak enabled
3. Deploys worker pods that join the mesh
4. Creates API key, deploys app via SSH over mesh
5. Verifies app is accessible through the mesh

### Chart Structure

```
charts/wonder-mesh-net/
├── Chart.yaml                    # Chart metadata
├── values.yaml                   # Default values
├── templates/
│   ├── deployment.yaml           # Coordinator + Headscale pod
│   ├── deployment-keycloak.yaml  # Optional Keycloak
│   ├── deployment-postgresql.yaml # Optional PostgreSQL
│   ├── configmap-headscale.yaml  # Headscale config
│   ├── configmap-keycloak.yaml   # Keycloak realm import
│   ├── secret.yaml               # JWT secret, API key
│   ├── ingress.yaml              # Optional Ingress
│   └── ...
```

## Architecture

```
cmd/wonder/
├── main.go              # CLI entry point (cobra/viper)
├── commands/
│   ├── coordinator.go   # Coordinator server command
│   └── worker/          # Worker CLI (join/status/leave)

internal/app/coordinator/
├── server.go            # HTTP server bootstrap and middleware
├── controller/          # HTTP handlers (thin layer)
├── service/             # Business logic
├── repository/          # Data access (uses sqlc queries)
├── database/            # DB connection, migrations (goose), sqlc queries
│   ├── goose/           # Migration files (modify 001_init.sql during dev)
│   └── sqlc/            # Query definitions + generated code
└── webui/               # Embedded static assets for React UI

pkg/
├── meshbackend/         # Backend interface + Tailscale implementation
│   └── tailscale/       # Headscale-based mesh backend
├── headscale/           # Headscale API client (wondernet, ACL)
├── jointoken/           # JWT-based join tokens for workers
├── jwtauth/             # JWT validation middleware
├── apikey/              # API key generation/validation
└── wondersdk/           # Client SDK for external integrations

webui/                   # React/TypeScript SPA (Vite)
charts/wonder-mesh-net/  # Helm chart for Kubernetes deployment

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/STRRL) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
