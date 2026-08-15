---
trigger: always_on
description: > This file is the primary entry point for AI agents working on ballast.
---

# ballast — Agent Guide

> This file is the primary entry point for AI agents working on ballast.
> Read DESIGN.md for full system context before making changes.
> **For any change to enrollment or WorkloadProfile lifecycle, read
> [docs/convergence.md](docs/convergence.md) first** — it is the canonical
> convergence contract (sequence diagrams + invariants) and must be updated
> alongside such changes.
> The Key Files table below is the current reference. IMPLEMENTATION_PLAN.md is a
> historical record of the phased build-out, not a live status tracker.

## Project Overview

Ballast is a Kubernetes operator that automatically right-sizes workload resource requests and limits based on real operational history. Workloads opt in via a single pod-template label, `ballast.tightlinesoftware.com/mode`; Ballast never touches a workload without it.

The operator has three main behaviors:
1. **Measure** — collect per-container CPU/memory usage samples into Redis time-series keys
2. **Apply** — patch resource requests/limits at admission time when a pod is admitted
3. **Resize** — adjust resources on running pods via the Kubernetes in-place resize API (1.35+)

Nothing is applied or resized until the matching `WorkloadProfile` has accumulated enough history and its `meetsThreshold` field is true. The `mode` label value picks a rung on an escalating ladder (`measure`, `apply`, or `resize`); each rung implies the ones below it, so `mode: resize` enables all three.

## Quick Start

```bash
make check        # Full gate: lint + coverage + build
make test         # Run tests
make build        # Build bin/ballastd
make fmt          # Format code
make lint-fix     # Auto-fix lint issues
make manifests    # Regenerate CRDs and RBAC from markers
make generate     # Regenerate DeepCopy methods
```

## Key Files

| File | Purpose |
|---|---|
| `cmd/ballastd/main.go` | Entry point; all CLI flags; creates kubebuilder manager; registers kill switch, all controllers, and webhook |
| `api/v1/ballastconfig_types.go` | `BallastConfig` CRD — cluster-scoped singleton; `identityLabels`, `orphanTTL`, `retentionWindow`, `suspended` |
| `api/v1/metricssource_types.go` | `MetricsSource` CRD — names a plugin type (`spec.type`) and its poll config |
| `api/v1/clusterresourcepolicy_types.go` | `ClusterResourcePolicy` CRD — selector, metrics slice, readiness config, behaviors |
| `api/v1/resourcepolicy_types.go` | `ResourcePolicy` CRD — same spec as `ClusterResourcePolicy`; namespace-scoped; always beats `ClusterResourcePolicy` |
| `api/v1/workloadprofile_types.go` | `WorkloadProfile` CRD — status-only; `tupleLabels`, `containers` (usageStats + recommendations), `meetsThreshold`, `activeWorkloads`, conditions |
| `internal/killswitch/killswitch.go` | `KillSwitch` reconciler; `IsActive()`/`Reason()` hot path; watches ConfigMap `ballast-kill-switch` and `BallastConfig.spec.suspended` |
| `internal/logger/logger.go` | `New(component, level, format) logr.Logger` backed by zap; `newWithWriter` is the testable variant |
| `internal/policy/resolver.go` | `Resolver`; `Resolve(ctx, Input) (*ResolvedPolicy, error)` — evaluates namespace/annotation/label selectors; `ResourcePolicy` beats `ClusterResourcePolicy` regardless of priority |
| `internal/store/client.go` | `Client` interface (go-redis subset: RPush, LTrim, LRange, LLen, SetNX, Get, Del, Scan); `NewClient(redisURL)` |
| `internal/store/keys.go` | `TupleHash(labels)`, `MetricKey(hash, container, resource)`, `AllKeysForHash(ctx, client, hash)` |
| `internal/store/metrics.go` | `AddSample`, `QueryAll`, `FirstSeenMs`, `SampleCount`, `DeleteKey` — list-based API; see Redis Data Model |
| `internal/store/percentiles.go` | `ComputeStats([]int64) Stats` — p50/p95/p99/max/mean/stddev/CV |
| `internal/plugin/plugin.go` | `MetricsPlugin` interface; `WorkloadIdentity`, `TimeWindow`, `ContainerStats` types |
| `internal/plugin/registry.go` | `Register(p)`, `Get(typeName)` — global plugin registry; plugins self-register via `init()` |
| `internal/plugin/kubernetes/plugin.go` | `kubernetesMetrics` plugin — calls in-cluster metrics API; token-bucket rate limiting; exponential backoff on errors |
| `internal/stats/aggregator.go` | `EvaluateReadiness(Stats, firstMs, lastMs, ReadinessConfig, resourceName) bool`; `ComputeRecommendation(Stats, MetricConfig) (resource.Quantity, error)` |
| `internal/validation/enrollment.go` | `LabelMode` + `Mode`/`IsEnrolled`/`WantsApply`/`WantsResize`/`ValidateMode` — the mode-label enrollment model |
| `internal/controller/workloadwatcher/controller.go` | Watches pods; creates/updates `WorkloadProfile`; `ProfileName(tupleLabels, identityLabels)` and `ExtractTupleLabels(podLabels, identityLabels)` exported for webhook use |
| `internal/controller/metricscollector/controller.go` | Reconciles `WorkloadProfile` on timer; polls plugins; writes to Redis lists; updates status with stats and recommendations |
| `internal/controller/resourceadjuster/controller.go` | Watches `WorkloadProfile` status changes; detects drift; issues in-place pod resize patches; exports `ExceedsDrift`, `CapChange`, `ResolveFieldThreshold`, `ParseResizeInterval` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Tight-Line/ballast](https://github.com/Tight-Line/ballast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
