---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **specification-driven project** — the repository centers on a single document (`SPIFFE-MCP-OAuth-KindCluster-Spec.md`, ~1400 lines) that provides a complete implementation guide for deploying a SPIFFE/SPIRE + Keycloak + MCP OAuth authentication system inside a Kind (Kubernetes-in-Docker) cluster on macOS.

The system authenticates MCP (Model Context Protocol) OAuth clients using cryptographically verifiable SPIFFE identities instead of static client secrets, following IETF draft `draft-ietf-oauth-spiffe-client-auth` and Christian Posta's reference implementation.

## Architecture

**Core components (all run inside a Kind cluster named `spiffe-mcp-demo`):**

- **SPIRE Server/Agent** (Helm chart, namespace `spire-system`): Issues short-lived JWT SVIDs to workloads. Trust domain: `example.org`
- **SPIFFE OIDC Discovery Provider**: Serves JWKS so Keycloak can validate JWT SVIDs
- **Keycloak** (namespace `mcp-demo`): OAuth2/OIDC Authorization Server with two custom SPIs:
  - `spiffe-svid-client-authenticator` (Java) — validates JWT SVIDs as client assertions
  - `spiffe-dcr-keycloak` (Java) — Dynamic Client Registration using SPIFFE software statements
- **SPIRE software-statements plugin** (Go) — CredentialComposer that enriches JWT SVIDs with DCR claims
- **MCP Client/Server pods** (Python 3.12, namespace `mcp-demo`): Demo workloads
- **OIDC HTTP Proxy** (nginx, namespace `spire-system`): Terminates TLS from the OIDC Discovery Provider so Keycloak can reach JWKS over plain HTTP inside the cluster
- **PostgreSQL**: Keycloak backend datastore
- Services exposed via **NodePort** + Kind `extraPortMappings` (no ingress controller or port-forward)

**Two authentication flows:**
1. **Flow A (DCR)**: MCP client gets JWT SVID → POSTs to Keycloak's SPIFFE DCR endpoint → auto-registers as OAuth client
2. **Flow B (Token exchange)**: MCP client uses JWT SVID as `client_assertion` (type `urn:ietf:params:oauth:client-assertion-type:jwt-spiffe`) instead of `client_secret` to get access tokens

**Dependency chain:** Kind cluster → SPIRE CRDs → SPIRE Stack → PostgreSQL → Keycloak → ClusterSPIFFEIDs → MCP workloads → test script

## Key External Dependencies

- `github.com/christian-posta/spiffe-svid-client-authenticator` (Java/Maven)
- `github.com/christian-posta/spiffe-dcr-keycloak` (Java/Maven)
- `github.com/christian-posta/spire-software-statements` (Go 1.21+)

## Implementation: Phased Execution

The spec is designed to be executed in **5 ordered phases**, each completable in one session:

| Phase | Name | Spec Sections | Checkpoint |
|-------|------|---------------|------------|
| 1 | Infrastructure | 3, 4 | Kind cluster running with NodePort mappings |
| 2 | SPIRE Stack | 5 | All SPIRE pods Running, healthcheck passing |
| 3 | Custom Builds | 6 | Docker images built and loaded into Kind |
| 4 | Keycloak + Data | 7, 8, 10 | Keycloak admin accessible, realm imported |
| 5 | Workloads + Validation | 5.5, 9, 13 | MCP pods running, test-spiffe-auth.sh passes |

**State tracking:** Use `implementation-state.json` to persist progress between sessions. At session start, read it and run the validation gate for the last completed phase before continuing.

## Common Commands

```bash
# Prerequisites check
docker version && kind version && kubectl version --client && helm version && jq --version

# Cluster lifecycle
kind create cluster --config kind-config.yaml
kind delete cluster --name spiffe-mcp-demo

# SPIRE deployment
helm repo add spiffe https://spiffe.github.io/helm-charts-hardened/ && helm repo update
helm upgrade --install spire spiffe/spire -n spire-system --create-namespace -f spire-values.yaml

# Load custom images into Kind
kind load docker-image keycloak-spiffe:latest --name spiffe-mcp-demo

# Full deploy/teardown (once files are extracted from spec)
./deploy-all.sh
./teardown.sh

# Configure Keycloak SPIFFE auth flow (after Keycloak is ready)
./scripts/configure-keycloak.sh

# Health check — quick pass/fail for all components
./scripts/continuous-validation.sh

# Run full test suite (executes inside test-runner pod)
./scripts/run-all-tests.sh

# Run a single test file inside the test-runner pod
kubectl -n mcp-demo exec deploy/test-runner -- python3 -m pytest /tests/test_01_infrastructure.py -v

# Run the SPIFFE auth demo (inside mcp-client pod)
kubectl -n mcp-demo exec deploy/mcp-client -- bash /scripts/test-spiffe-auth.sh
```

## Operational Guidelines

**Context management:** Do NOT read the entire spec at once. Load only the current phase's sections using targeted line ranges.

**Command output:** Truncate Kubernetes output aggressively — use `| head -60`, `--tail=30`, `-o wide`, and `| grep <key>` instead of dumping full resources.

**Idempotency:** Always use `kubectl apply` (never `create`), `helm upgrade --install`, and `kind get clusters | grep -q ... || kind create cluster`.

**Error diagnosis — tiered approach (do not blindly retry):**
1. `kubectl get pods -n <ns> | grep -v Running`
2. `kubectl get events -n <ns> --sort-by=.lastTimestamp | tail -10`
3. `kubectl logs <pod> --tail=30 -n <ns>`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [a2z-ice/AAuthSpiffe](https://github.com/a2z-ice/AAuthSpiffe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
