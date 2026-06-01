---
trigger: always_on
description: This file provides guidance to Codex, Cursor, Copilot, and other coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex, Cursor, Copilot, and other coding agents when working with code in this repository.

<!-- AUTO-SYNCED: canonical source is .claude/CLAUDE.md. Only the first 5 lines differ. -->

## 1. Project Overview and Architecture

Topograph discovers the physical network topology of a cluster (NVLink domains, InfiniBand/Ethernet switch fabric, cloud rack topology) and exposes it to workload schedulers — Slurm, Kubernetes, and Slurm-on-Kubernetes (Slinky). It has five runtime components:

- **API Server** — receives `/v1/generate` requests, aggregates bursts, dispatches to a Provider
- **Node Observer** — Kubernetes-only; watches node status changes and triggers regeneration
- **Node Data Broker** — Kubernetes-only DaemonSet; collects per-node attributes (NVLink clique IDs, etc.) as node annotations
- **Provider** — per-environment adapter that queries a topology source (CSP API, NetQ, `ibnetdiscover`, DRA labels) and returns a canonical representation
- **Engine** — per-scheduler translator that writes the canonical representation out as `topology.conf`, Kubernetes node labels, or a Slinky ConfigMap

### Key invariant

Providers differ by environment. The canonical `topology.Graph` is stable. Engines only translate — they do not discover.

This separation is load-bearing. If you find yourself reading the fabric in an engine, or emitting scheduler-specific output from a provider, stop and reconsider.

### Repository map

```
cmd/                  # Four entry points: topograph, node-observer, node-data-broker-initc
pkg/
  providers/          # One directory per provider: aws, gcp, oci, nebius, netq, dra, infiniband, lambdai, cw, test
  engines/            # One directory per engine: k8s, slinky, slurm
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
charts/topograph/     # Helm chart (with node-data-broker subchart)
docs/                 # Public-facing docs — overview.md, architecture.md, api.md + providers/, engines/, reference/ subdirectories
tests/models/         # YAML simulation fixtures
tests/charts/         # Helm golden outputs for chart values fixtures
config/               # Sample topograph-config.yaml
scripts/              # Build scripts (deb, rpm, SSL, clean)
localdev/             # Developer-local workspace — not tracked; personal scratch files
```

### Do not change without discussion

These structures propagate across every provider and engine. Changing them in a single PR usually means the PR is too broad.

| Surface | Why it's load-bearing |
|---|---|
| `pkg/topology/` — `Graph`, the `Vertex` tree, and topology constants | Every provider returns it; every engine consumes it. A shape change ripples to all of them. |
| Helm `global.provider.name` / `global.engine.name` / `topologyNodeLabels` | External contract for operators deploying Topograph. |
| The four default label keys `network.topology.nvidia.com/{accelerator,leaf,spine,core}` | Consumed by downstream projects (KAI Scheduler, NVSentinel, Kueue). |

## 2. Setup and Installation

### Prerequisites

- **Go 1.25.9** (see `go.mod`) — newer minor versions are fine; older will not build
- **make**
- **golangci-lint** — `brew install golangci-lint` or via `go install`
- **docker** — only for container image builds and the IB variant

### Clone and build

```bash
git clone https://github.com/NVIDIA/topograph.git
cd topograph
make build   # produces bin/topograph, bin/node-observer, bin/node-data-broker-initc
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
make chart-test                 # helm chart smoke + golden tests (see scripts/chart-test.sh)
make chart-test-update-golden   # refresh tests/charts/*.golden.yaml (review before commit)
make coverage   # human-readable per-package summary
```

Run `make qualify` before pushing. The individual targets are available if you want to run a single check during iteration. Run `make chart-test` when you change `charts/topograph/` or its subcharts; CI runs it on every workflow trigger.

### Coverage policy

From `codecov.yml`:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA/topograph](https://github.com/NVIDIA/topograph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
