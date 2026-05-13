---
trigger: always_on
description: Prometheus exporter that monitors Kubernetes binpacking efficiency. Compares pod resource requests against node allocatable capacity using informer-based caching (zero API calls per scrape).
---

# CLAUDE.md

## Project Overview

Prometheus exporter that monitors Kubernetes binpacking efficiency. Compares pod resource requests against node allocatable capacity using informer-based caching (zero API calls per scrape).

**Purpose**: Helps identify scheduling inefficiency by showing how well pods are packed onto nodes. High allocatable with low allocated means wasted capacity. High allocated means good utilization.

## Architecture

- **Flat layout**: 3 Go files in `package main` at root — no `pkg/`, `internal/`, `cmd/`
- **Plain client-go + informers**: No controller-runtime. This is an exporter, not a controller
- **Resource-agnostic**: Metrics use a `resource` label. Adding GPU/ephemeral-storage is config-only (`--resources`)
- **Scrape-time computation**: `MustNewConstMetric` in `Collect()` — avoids stale metrics for removed nodes
- **Cardinality control**: Optional `--disable-node-metrics` to reduce metric count by 90%+ in large clusters
- **Init container aware**: Correctly accounts for init containers using Kubernetes scheduler semantics

## File Map

| File | Role | Key Functions |
|------|------|---------------|
| `main.go` | Entry point, HTTP server | Flag parsing, signal handling, `/metrics`, `/healthz`, `/readyz`, `/sync` endpoints |
| `kubernetes.go` | Kube client setup | `setupKubernetes()` - config resolution, informer factories (node + pod), cache sync with progress logging |
| `collector.go` | Prometheus collector | `Collect()` - computes metrics, `calculatePodRequest()` - init container logic |
| `Dockerfile` | Container image | Multi-stage: `golang:1.25-alpine` → `distroless/static-debian12:nonroot` |
| `chart/` | Helm chart | RBAC, ServiceMonitor, published to OCI registry at ghcr.io |
| `.github/workflows/` | CI/CD | `ci.yaml` - build/vet/lint/test, `release.yaml` - PR merge → semver tag → GoReleaser + Docker + Helm OCI push (single workflow) |

## Metrics Exported

All metrics computed at scrape time from informer cache:

| Metric | Type | Labels | Purpose |
|--------|------|--------|---------|
| `kube_binpacking_node_allocated` | Gauge | `node`, `resource` | Total resource requests on node |
| `kube_binpacking_node_allocatable` | Gauge | `node`, `resource` | Node capacity |
| `kube_binpacking_node_utilization_ratio` | Gauge | `node`, `resource` | allocated / allocatable (0.0-1.0+) |
| `kube_binpacking_node_daemonset_overhead` | Gauge | `node`, `resource` | DaemonSet resource requests on node |
| `kube_binpacking_node_daemonset_overhead_ratio` | Gauge | `node`, `resource` | DaemonSet overhead / allocatable (0.0-1.0+) |
| `kube_binpacking_cluster_allocated` | Gauge | `resource` | Cluster-wide total requests |
| `kube_binpacking_cluster_allocatable` | Gauge | `resource` | Cluster-wide capacity |
| `kube_binpacking_cluster_utilization_ratio` | Gauge | `resource` | Cluster-wide ratio |
| `kube_binpacking_cluster_daemonset_overhead` | Gauge | `resource` | Cluster-wide DaemonSet requests |
| `kube_binpacking_cluster_daemonset_overhead_ratio` | Gauge | `resource` | Cluster-wide DaemonSet overhead ratio |
| `kube_binpacking_cluster_node_count` | Gauge | - | Total number of nodes in cluster |
| `kube_binpacking_group_allocated` | Gauge | `label_group`, `label_group_value`, `resource` | Total requests on nodes in this label group (only if `--label-group` set) |
| `kube_binpacking_group_allocatable` | Gauge | `label_group`, `label_group_value`, `resource` | Total capacity on nodes in this label group (only if `--label-group` set) |
| `kube_binpacking_group_utilization_ratio` | Gauge | `label_group`, `label_group_value`, `resource` | Ratio for nodes in this label group (only if `--label-group` set) |
| `kube_binpacking_group_daemonset_overhead` | Gauge | `label_group`, `label_group_value`, `resource` | DaemonSet requests on nodes in this label group (only if `--label-group` set) |
| `kube_binpacking_group_daemonset_overhead_ratio` | Gauge | `label_group`, `label_group_value`, `resource` | DaemonSet overhead ratio for this label group (only if `--label-group` set) |
| `kube_binpacking_group_node_count` | Gauge | `label_group`, `label_group_value` | Node count for this label group (only if `--label-group` set) |
| `kube_binpacking_cache_age_seconds` | Gauge | - | Time since last informer sync |

## HTTP Endpoints

| Endpoint | Purpose | Returns |
|----------|---------|---------|
| `/` | Homepage | HTML page with links to all endpoints and configuration |
| `/metrics` | Prometheus scrape target | Text exposition format |
| `/healthz` | Liveness probe | 200 if process alive |
| `/readyz` | Readiness probe | 200 if cache synced, 503 otherwise |
| `/sync` | Cache status | JSON with last sync time, age, resync period, sync state |

## Build & Verify

```bash
# Build
go build -o kube-binpacking-exporter .

# Verify
go vet ./...
helm lint chart

# Run locally
go run . --kubeconfig ~/.kube/config
go run . --kubeconfig ~/.kube/config --log-level=debug                                          # verbose
go run . --resync-period=1m --log-level=debug                                                   # fast resync

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [procore-oss/kube-binpacking-exporter](https://github.com/procore-oss/kube-binpacking-exporter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
