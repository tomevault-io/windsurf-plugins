---
trigger: always_on
description: Guidance for AI coding agents working in this repository. Human contributors should also read [CONTRIBUTING.md](CONTRIBUTING.md) and the [documentation site](https://dra-driver-nvidia-gpu.sigs.k8s.io/), which are the source of truth.
---

# AGENTS.md

Guidance for AI coding agents working in this repository. Human contributors should also read [CONTRIBUTING.md](CONTRIBUTING.md) and the [documentation site](https://dra-driver-nvidia-gpu.sigs.k8s.io/), which are the source of truth.

## Project summary

This is the **DRA Driver for NVIDIA GPUs**, a Kubernetes [Dynamic Resource Allocation](https://kubernetes.io/docs/concepts/scheduling-eviction/dynamic-resource-allocation/) driver. It is a SIG Node subproject (`sigs.k8s.io/dra-driver-nvidia-gpu`) and follows Kubernetes / `client-go` conventions. It manages two kinds of resources, each with its own kubelet plugin:

- **`ComputeDomain`s** (`cmd/compute-domain-controller/`, `cmd/compute-domain-daemon/`, `cmd/compute-domain-kubelet-plugin/`) — an abstraction for robust, secure Multi-Node NVLink (MNNVL). Officially supported. Orchestrates IMEX daemons, domains, and channels under the hood.
- **`GPU`s** (`cmd/gpu-kubelet-plugin/`) — flexible allocation and dynamic reconfiguration of GPUs (sharing, MIG, time-slicing). Officially supported, but some features are still experimental (alpha).

API group: `resource.nvidia.com/v1beta1` (`api/nvidia.com/resource/v1beta1/`). Go module: `sigs.k8s.io/dra-driver-nvidia-gpu` (Go 1.26.x, see [go.mod](go.mod)).

## AI Contribution Policy

This project follows the [Kubernetes AI Tool Usage Policy](https://www.kubernetes.dev/docs/guide/pull-requests/#ai-guidance). Key rules:

- **Disclose AI usage** in the PR description (use `#### Special notes for your reviewer:` in [.github/PULL_REQUEST_TEMPLATE.md](.github/PULL_REQUEST_TEMPLATE.md)).
- **Disclose AI usage** when commenting or filing issues.
- **No AI authorship markers.** Do not add AI co-author lines, `assisted-by`, `co-developed`, or similar commit trailers.
- All PRs still require a signed [CNCF CLA](https://git.k8s.io/community/CLA.md) regardless of how the change was produced.

## Repository layout

| Path | What lives there |
| --- | --- |
| [api/nvidia.com/resource/v1beta1/](api/nvidia.com/resource/v1beta1/) | `ComputeDomain`, `GpuConfig`, `ComputeDomainConfig`, `MigConfig`, `VfioDeviceConfig`, sharing/IOMMU types, and their validation (`validate.go`). Hand-written except `zz_generated.deepcopy.go`. |
| [cmd/gpu-kubelet-plugin/](cmd/gpu-kubelet-plugin/) | GPU kubelet plugin: NVML/nvidia-smi device discovery, MIG, sharing, time-slicing. |
| [cmd/compute-domain-kubelet-plugin/](cmd/compute-domain-kubelet-plugin/) | ComputeDomain kubelet plugin: node-local IMEX channel/domain setup. |
| [cmd/compute-domain-controller/](cmd/compute-domain-controller/), [cmd/compute-domain-daemon/](cmd/compute-domain-daemon/) | Cluster-level ComputeDomain reconciliation and the per-node IMEX daemon. |
| [cmd/webhook/](cmd/webhook/) | Admission webhook validating opaque device config in `ResourceClaim`/`ResourceClaimTemplate` across `resource.k8s.io` v1, v1beta1, v1beta2. |
| [templates/](templates/) | Go `text/template` files rendered at runtime by the controller/kubelet-plugin binaries (e.g. the per-ComputeDomain IMEX DaemonSet, claim templates) — not part of the Helm chart, no `values.yaml` override path. |
| [pkg/nvidia.com/](pkg/nvidia.com/) | **Generated** typed clientset, informers, listers for the `resource.nvidia.com` API group. Do not hand-edit. |
| [pkg/featuregates/](pkg/featuregates/), [pkg/flags/](pkg/flags/) | Feature gate definitions and CLI flag/leader-election/logging helpers shared across binaries. |
| [pkg/imex/](pkg/imex/), [pkg/fabricmanager/](pkg/fabricmanager/) | Clients for IMEX and NVIDIA Fabric Manager, used by the ComputeDomain path. |
| [pkg/metrics/](pkg/metrics/) | Prometheus metric definitions (ComputeDomain cluster state, DRA request counters). |
| [pkg/bootid/](pkg/bootid/), [pkg/flock/](pkg/flock/), [pkg/workqueue/](pkg/workqueue/) | Small shared internals (node boot-ID detection, file locking, a jitter-limited workqueue). |
| [test/e2e/](test/e2e/) | Go/Ginkgo e2e suite (`//go:build e2e`), run via `make test-e2e` against a live cluster with real GPUs. |
| [tests/bats/](tests/bats/) | Bats integration suites against real GPU/MNNVL hardware (`test_gpu_*.bats`, `test_cd_*.bats`). Run via Prow, plus a mock-NVML subset on GitHub Actions ([.github/workflows/mock-nvml-e2e.yaml](.github/workflows/mock-nvml-e2e.yaml)). |
| [deployments/helm/dra-driver-nvidia-gpu/](deployments/helm/dra-driver-nvidia-gpu/) | Helm chart, including the CRDs mirrored under its `crds/` directory. |
| [deployments/devel/](deployments/devel/) | Pinned dev toolchain (golang version, codegen tool versions) and `check-modules`. |
| [hack/](hack/) | Codegen boilerplate, CI scripts, image build/publish scripts. |
| [site/content/](site/content/) | Hugo docs source for https://dra-driver-nvidia-gpu.sigs.k8s.io. `site/content/contribute/proposals/` holds design proposals (this project's KEP-equivalent). |
| [demo/](demo/) | Runnable sample manifests for the kind-based quickstart demo. |

When in doubt about ownership, check [OWNERS](OWNERS).

## Build, test, lint


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kubernetes-sigs/dra-driver-nvidia-gpu](https://github.com/kubernetes-sigs/dra-driver-nvidia-gpu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
