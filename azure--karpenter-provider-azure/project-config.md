---
trigger: always_on
description: This repository is the AKS Karpenter Provider: the Azure cloud provider implementation for
---

# Overview

This repository is the AKS Karpenter Provider: the Azure cloud provider implementation for
[Karpenter](https://karpenter.sh/). It watches for unschedulable pods, evaluates their scheduling
constraints, provisions Azure nodes that satisfy them, and removes or consolidates nodes when they
are no longer needed.

The same codebase serves two deployment models, described in [README.md](./README.md):

- **Node Auto Provisioning (NAP)** — Karpenter runs as an AKS-managed addon. AKS manages token
  rotation, Helm charts, Karpenter version updates, VM OS disk updates, and Linux node image
  upgrades.
- **Self-hosted** — Karpenter runs as a standalone deployment in the cluster. The user manages
  upgrades, token rotation, and Helm charts directly.

Images are built differently for the two models and some code paths diverge, but the source is
shared. A change that is correct for one model is not automatically correct for the other.

## Provisioning Modes

The provider supports several provisioning modes, defined in
[`pkg/consts/consts.go`](./pkg/consts/consts.go) and selected via the `--provision-mode` flag /
`PROVISION_MODE` environment variable:

- `aksscriptless`
- `bootstrappingclient`
- `aksmachineapi`
- `aksmachineapiheaderbatch`

These fall into two families with materially different implementations:

- **VM-based** (`aksscriptless`, `bootstrappingclient`) — the provider creates and manages Azure
  VMs, NICs, and extensions directly. See
  [`pkg/providers/instance/vminstance.go`](./pkg/providers/instance/vminstance.go).
- **AKS Machine API-based** (`aksmachineapi`, `aksmachineapiheaderbatch`) — the provider creates AKS
  Machine resources and AKS performs the underlying compute operations. See
  [`pkg/providers/instance/aksmachineinstance.go`](./pkg/providers/instance/aksmachineinstance.go)
  and `aksmachineinstancehelpers.go`.

The two AKS Machine API modes are **the same provisioning path**, not separate implementations.
Both use `DefaultAKSMachineProvider`, the same machine template construction, and the same
lifecycle, read, update, and delete logic. `aksmachineapiheaderbatch` differs only in how creates
are dispatched: compatible concurrent creates are accumulated and sent as a single request carrying
a shared machine template plus a `BatchPutMachine` header describing each machine, then polled
individually via GET. See
[`pkg/providers/azclient/aksmachinesheaderbatch/`](./pkg/providers/azclient/aksmachinesheaderbatch/)
and the create-path branch in `aksmachineinstance.go`. Treat them as one mode with two create
dispatch strategies — a behavioral change generally applies to both, and batching concerns are
about grouping, partial failure, and batch limits rather than about a different machine model.

Several files carry explicit `ATTENTION!!!` comments stating that changes there may not take effect
on AKS machine nodes, and point at the counterpart implementation. Treat those comments as
authoritative — they mark the exact places where a one-sided change silently breaks a mode.

## Code Structure

- [`pkg/apis/`](./pkg/apis/) — `AKSNodeClass` API types. `v1beta1` is the current version;
  `v1alpha2` is deprecated and planned for removal. Also holds generated deepcopy code and CRDs
  under `pkg/apis/crds/`.
- [`pkg/cloudprovider/`](./pkg/cloudprovider/) — implementation of the upstream Karpenter
  `CloudProvider` interface (create, delete, get, list, drift).
- [`pkg/controllers/`](./pkg/controllers/) — provider-specific controllers: nodeclaim
  garbage collection and in-place update; nodeclass hash, status, and termination; instance type
  and quota controllers.
- [`pkg/providers/`](./pkg/providers/) — Azure integrations: `azclient`, `instance`, `instancetype`,
  `imagefamily`, `launchtemplate`, `kubernetesversion`, `loadbalancer`,
  `networksecuritygroup`, `pricing`, `quota`, `zone`, `allocationstrategy`, `labels`.
- [`pkg/operator/`](./pkg/operator/) — operator wiring and options/flags.
- [`pkg/fake/`](./pkg/fake/) and [`pkg/test/`](./pkg/test/) — fakes and the test environment used by
  acceptance tests.
- [`charts/`](./charts/) — `karpenter` and `karpenter-crd` Helm charts. CRD templates are copied
  from `pkg/apis/crds` by `make verify`.
- [`test/`](./test/) — real end-to-end suites run against a live AKS cluster. See
  [test/README.md](./test/README.md).
- [`hack/`](./hack/) — toolchain, codegen, validation, and release scripts.

## Development and Validation

The supported entry points are the Make targets; see [CONTRIBUTING.md](./CONTRIBUTING.md).

| Target | Purpose |
|--------|---------|
| `make verify` | Codegen, boilerplate, CRD copy, validation, linting, `actionlint`. Run after any code change. |
| `make presubmit` | `make verify` plus tests. Run before submitting. |
| `make test` | Unit and acceptance tests via Ginkgo. |
| `make e2etests` | E2E suites against a live cluster (`TEST_SUITE=<suite>`). |
| `make toolchain` | Install the correct tool versions; fixes most local environment failures. |

Do not run `go generate` directly and do not hand-edit generated output. `make verify` performs
code generation, copies upstream and provider CRDs into `pkg/apis/crds` and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure/karpenter-provider-azure](https://github.com/Azure/karpenter-provider-azure) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
