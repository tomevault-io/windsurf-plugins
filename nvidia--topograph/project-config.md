---
trigger: always_on
description: This file provides guidance to Claude Code when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

<!-- This is the canonical source. AGENTS.md is a synced public copy; only the first 5 lines differ. -->

## 1. Project Overview and Architecture

Topograph discovers the physical network topology of a cluster (NVLink domains, InfiniBand/Ethernet switch fabric, cloud rack topology) and exposes it to workload schedulers — Slurm, Kubernetes, and Slurm-on-Kubernetes (Slinky). It has five runtime components:

- **API Server** — receives `/v1/generate` requests, aggregates bursts, dispatches to a Provider
- **Node Observer** — Kubernetes-only; watches configured node/pod changes and Topograph API readiness, then triggers regeneration
- **Node Data Broker** — Kubernetes-only DaemonSet; collects per-node attributes (NVLink clique IDs, etc.) as node annotations
- **Provider** — per-environment adapter that queries a topology source (CSP API, NetQ, `ibnetdiscover`, DRA labels) and returns a canonical representation
- **Engine** — per-scheduler translator that writes the canonical representation out as `topology.conf`, Kubernetes node labels, or a Slinky ConfigMap

### Key invariant

Providers differ by environment. The canonical `topology.Graph` is stable. Engines only translate — they do not discover.

This separation is load-bearing. If you find yourself reading the fabric in an engine, or emitting scheduler-specific output from a provider, stop and reconsider.

### Repository map

```
cmd/                  # Entry points: topograph, node-observer, node-data-broker, kwok-nodes
pkg/
  providers/          # One directory per provider: aws, gcp, oci, nebius, netq, dra, infiniband, lambdai, test
  engines/            # One directory per engine: k8s, nfd, slinky, slurm
  topology/           # Canonical Graph, Vertex tree, and topology constants (DO NOT CHANGE CASUALLY)
  registry/           # Central NamedLoader wiring for providers + engines
  translate/          # topology.conf and block/tree generation shared by engines
  server/             # HTTP server and request aggregator
  node_observer/      # Kubernetes Node watcher
  ib/                 # InfiniBand fabric discovery helpers
  config/             # Config file parser
  metrics/            # Prometheus metrics
  models/             # Go types and loader for YAML simulation models (the YAML files live in tests/models/)
  test/               # Cross-package test helpers
internal/             # Shared utilities not part of the public API
  cluset, component, config, exec, files, httperr, httpreq, k8s, version
charts/topograph/     # Helm chart for all Kubernetes components; tests/ holds the helm-unittest suites + snapshots
CHANGELOG.md          # Release history (Keep a Changelog format); update [Unreleased] for user-facing PRs
docs/                 # Public-facing docs — overview.md, architecture.md, api.md + providers/, engines/, reference/ subdirectories
demos/                # Interactive Kubernetes/KWOK deployment demos
tests/models/         # YAML simulation fixtures
config/               # Sample topograph-config.yaml
scripts/              # Build scripts (deb, rpm, SSL, clean)
localdev/             # Developer-local workspace — not tracked; personal scratch files
```

### Do not change without discussion

These structures propagate across every provider and engine. Changing them in a single PR usually means the PR is too broad.

| Surface | Why it's load-bearing |
|---|---|
| `pkg/topology/` — `Graph`, the `Vertex` tree, and topology constants | Every provider returns it; every engine consumes it. A shape change ripples to all of them. |
| Helm `provider.name` / `engine.name` | External contract for operators deploying Topograph. |
| The variable fabric labels `network.topology.nvidia.com/tier-N` and single accelerator label `network.topology.nvidia.com/accelerator` | Consumed by downstream projects (KAI Scheduler, NVSentinel, Kueue); fabric tier 0 is closest to the node. |

## 2. Setup and Installation

### Prerequisites

- **Go 1.26.5** (see `go.mod`) — newer minor versions are fine; older will not build
- **make**
- **golangci-lint** — `brew install golangci-lint` or via `go install`
- **helm 3.10+ or 4.x** — required for `make chart-test`; the `helm-unittest` plugin is installed automatically by the target (`brew install helm`). CI pins helm `v4.1.1` in `.github/workflows/chart-test.yaml`.
- **docker** — for container image builds (the main image includes `rdma-core` / `ibnetdiscover` for InfiniBand deployments)

### Clone and build

```bash
git clone https://github.com/NVIDIA/topograph.git
cd topograph
make build   # produces bin/topograph, bin/node-observer, bin/node-data-broker, bin/kwok-nodes
```

Cross-compile with `make build-linux-amd64`, `make build-darwin-arm64`, etc.

## 3. Testing and Deployment Workflows

### Local test loop

```bash
make qualify    # runs fmt, vet, lint, and test in sequence — pre-push aggregator
make fmt        # go fmt ./...
make vet        # go vet ./...
make lint       # golangci-lint run (only flags new issues vs. main)
make test       # go test -race -coverprofile=coverage.out ./...
make chart-test                  # helm lint + helm-unittest suites (charts/topograph/tests/)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA/topograph](https://github.com/NVIDIA/topograph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
