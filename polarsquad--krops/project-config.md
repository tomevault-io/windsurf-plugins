---
trigger: always_on
description: Guidance for AI coding agents working in this repository.
---

# AGENTS.md: krops

Guidance for AI coding agents working in this repository.

## What this repo is

A working reference implementation of a GitOps pattern for managing AWS
infrastructure through the Kubernetes API: no Terraform, no state files, no
second toolchain. A local kind cluster bootstraps Flux, which reconciles
everything else: CAPA-managed EKS workload clusters, per-cluster Flux
instances (CAPI addons), and ACK operators (S3, RDS, IAM) managing cloud
resources. There is no app source code here, only declarative infrastructure.

- `mgmt/aws/`: synced by the MANAGEMENT cluster's Flux.
  - `infrastructure/`: cert-manager, CAPI operator, CAPA identity, ACK
    controllers (S3, RDS, IAM), the per-cluster Bucket/DBInstance/reader
    Role CRs (`workload-resources/`), account-global IAM, konflate.
  - `capi-providers/`: capi-system, capa-system, caaph-system.
  - `addons/flux-apps/`: installs Flux on each workload cluster
    (HelmChartProxy + ClusterResourceSets).
  - `clusters/`: EKS cluster definitions per region (`eu-north-1`,
    `eu-west-1`); `eu-north-1` also defines the self-managed management
    cluster (`clusters/management/`).
- `mgmt/local-host/`: the local-host management variant (kind-based).
  Same layout as `mgmt/aws/` (`clusters/docker`, `capi-providers/`,
  `addons/`, `infrastructure/`) with no cloud dependencies.
- `mgmt/local-talos/`: the single-node Talos management variant (issue
  #105). Same component layout as `mgmt/aws/` minus addons
  (`infrastructure/`, `capi-providers/`, `clusters/management/`), synced
  from the GitHub GitRepository source like `mgmt/aws`, NOT the laptop OCI
  registry (a physical machine cannot reach krops-registry). Providers are
  Talos + Tinkerbell (CABPT/CACPPT from sidero-community releases, CAPT)
  instead of CAPD; the cluster definition is imperative (explicit
  controlPlaneRef, no ClusterClass) with committed site-specific values
  (control plane endpoint IP, Tinkerbell Hardware name). CAPT is pinned to
  the shrinedogg fork release v0.7.1 (upstream main plus the
  installer-image annotation mirror, PR tinkerbell#604; see
  `capi-providers/capt-system/provider.yaml`); re-point at upstream once a
  release there includes it. The installer image is declared on the
  TalosConfig through `spec.imageFactory` (CABPT v0.8.x resolves it against
  the Image Factory API); the committed definition declares no block, so no
  override is rendered, and the CAPT fork's annotation mirror is no longer
  consumed by CABPT v0.8.x. The `spec.imageFactory` path is not yet
  validated live: the #105 hardware acceptance run (done, closed) predates
  the CABPT bump to v0.8.2, and the PXE/Tinkerbell-Workflow provisioning
  transport still needs a run (issue #225). Fork retirement is tracked in
  issue #266, blocked on upstream PR
  tinkerbell/cluster-api-provider-tinkerbell#604. Scope fence:
  management-only; no `addons/` (Talos ships its own CNI, no
  HelmChartProxy consumers). The wiring landed in #169 and the docs in
  #171; the remaining #105 item is the hardware acceptance run.
- `mgmt/azure/`: the Azure management variant (issue #71). Same component
  layout as `mgmt/aws/` (`infrastructure/`, `capi-providers/`, `addons/`,
  `clusters/`), synced from GitHub. CAPZ v1.27.0 (`capi-providers/capz-system/`)
  bundles Azure Service Operator (ASO) into `capz-system`; clusters are
  `AzureASOManaged*` (AKS) with the ASO resources inline. The bundled ASO also
  reconciles the identity plumbing in `infrastructure/aso-workload-identity/`
  (user-assigned identities, per-cluster data resource group, role assignment,
  federated credentials), which replaces ACK pod identity. Credentials: none at
  rest; workload identity via the `krops-capz` UAMI (AzureClusterIdentity
  `type: WorkloadIdentity` + `credential-from` aso-credentials, a plain
  Secret). Federation is set up by the `arc-federate` mise task
  (`post-kind-create-task`): Arc OIDC issuer for kind, then the management
  cluster's own OIDC issuer post-pivot (issue #236). Non-secret IDs live in
  `azure-vars` (flux-system) and in the workload `cluster-vars`. Upgrade CAPZ
  one minor at a time (ASO CRD migrations). Teardown is manual until the live
  acceptance run.
- `mgmt/gcp/`: the GCP management variant (issue #72). Same component
  layout as `mgmt/aws/` (`infrastructure/`, `capi-providers/`, `addons/`,
  `clusters/`), synced from GitHub. CAPG v1.13.1
  (`capi-providers/capg-system/`) provisions GKE clusters
  (`GCPManaged*`); the Config Connector operator ships as a pinned verbatim
  release bundle (`infrastructure/kcc-operator/`, version comment
  `kcc-operator-version:`) and `tests/test-kcc-operator-pin.py` (in
  `mise run validate` and CI) keeps every committed copy byte-identical and
  matching the version comment: Renovate bumps the comment and the operator
  image tag, and the gate then goes red until the whole release bundle is
  re-downloaded (same "Renovate opens, human completes" posture as the
  Talos images). Credentials: none at rest; Workload Identity Federation
  through the `krops` pool with plain `external_account` Secrets
  (`capg-wif-credentials`, `kcc-wif-credentials`), provider
  `${GCP_WIF_PROVIDER:=mgmt}` (kind bootstrap overrides to `kind` via the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [polarsquad/krops](https://github.com/polarsquad/krops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
