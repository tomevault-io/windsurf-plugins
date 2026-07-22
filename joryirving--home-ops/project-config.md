---
trigger: always_on
description: This is a **Home Kubernetes cluster monorepo** managed with GitOps (Flux, Renovate, GitHub Actions).
---

# Home Operations - AI Assistant Guide

This is a **Home Kubernetes cluster monorepo** managed with GitOps (Flux, Renovate, GitHub Actions).

## Repository Structure

```
home-ops/
├── .agents/             # AI instructions & skills
│   ├── instructions/    # PR review system prompt, YAML sorting rules
│   └── skills/          # Reusable agent skills (e.g. add-app)
├── .github/             # GitHub Actions workflows & evidence providers
├── .renovate/           # Local Renovate config presets
├── .taskfiles/          # Task (taskfile.dev) operational commands
├── bootstrap/           # Bootstrap templates (helmfile, minijinja)
├── docs/                # mdBook documentation
├── hack/                # Operational scripts (see hack/README.md)
├── kubernetes/          # Kubernetes configurations (Flux-managed)
│   ├── apps/            # Application configs
│   │   ├── base/        # Shared base configs
│   │   ├── main/        # Main cluster overlay
│   │   ├── utility/     # Utility cluster overlay
│   │   └── test/        # Test cluster overlay
│   ├── clusters/        # Flux cluster definitions
│   └── components/      # Reusable k8s components
├── talos/               # Talos Linux machine configs
└── terraform/           # OpenTofu/Terraform IaC (cloud infra)
```

## Cluster Architecture

- **main** - 3x MS-01 + 1x Bosgame M5 (i9-13900H x3, Ryzen AI Max+ 395 x1, 128GB RAM), hyper-converged storage
- **utility** - 1x Bosgame P1 (Ryzen 7 5700U), low-power services
- **test** - 1x Beelink Mini-S (Celeron N5095), testing

## Key Technologies

| Category   | Tool                         | Purpose                                                                         |
| ---------- | ---------------------------- | ------------------------------------------------------------------------------- |
| GitOps     | Flux + flux-operator         | Deploys configs from Git to k8s; flux-operator manages the Flux instance itself |
| CI         | Renovate + GitHub Actions    | Dependency updates, automation                                                  |
| Networking | cilium (eBPF)                | CNI, BGP, service mesh                                                          |
| Ingress    | Envoy Gateway                | L7 proxy, ingress controller                                                    |
| DNS        | external-dns                 | Syncs ingress to Cloudflare/UniFi                                               |
| TLS        | cert-manager                 | TLS certificate automation                                                      |
| Secrets    | external-secrets + 1Password | Secret management                                                               |
| Storage    | Rook/Ceph + volsync          | Distributed storage + backups                                                   |
| Images     | spegel                       | Local OCI mirror                                                                |
| IaC        | tofu-controller              | Terraform on k8s                                                                |
| Charts     | app-template (bjw-s)         | Common Helm chart used by most apps                                             |
| Sources    | OCIRepository                | Flux source for OCI Helm charts (preferred)                                     |
| Reviews    | konflate                     | Rendered-diff evidence provider for PR reviews                                  |

## GitOps Flow

```
Git push → Flux source sync → Kustomization → HelmRelease → k8s resources
```

Flux recursively searches `kubernetes/${cluster}/apps/` for `kustomization.yaml` files. Each must define a namespace and Flux kustomization (`ks.yaml`).

## Conventions

- Component READMEs stay with components (e.g., `kubernetes/apps/base/cilium/README.md`)
- Secrets stored in 1Password, referenced via `external-secrets`
- SOPS used for encrypting sensitive values in Git
- Apps use `HelmRelease` via Flux, rarely raw manifests
- Clusters are mostly identical except for app selections and sizing
- **AI instructions**: `.agents/instructions/pr-review.instructions.md` is the live system prompt for the AI PR reviewer. `.agents/instructions/sorting.instructions.md` defines YAML sorting rules (including `app-template`-specific ordering). When editing YAML, follow the sorting instructions.
- **Namespace component**: `kubernetes/components/namespace/` injects the Namespace resource and alerting rules into every app via kustomize components. Helm chart sources are per-app: each app declares its own `OCIRepository` in `ocirepository.yaml`.
- **Namespace replacement**: `kubernetes/components/replacements/ks.yaml` propagates `spec.targetNamespace` into Flux Kustomizations automatically.

## Common Operations

- **Add app**: Create in `kubernetes/apps/${cluster}/` with kustomization + HelmRelease
- **Update app**: Merge renovate PR or manually edit and push
- **Troubleshoot**: Check `flux get all -n <namespace>`, `kubectl get events --sort-by=.lastTimestamp`
- **Scripts**: `hack/` contains operational scripts. See `hack/README.md` for the full list and usage.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joryirving/home-ops](https://github.com/joryirving/home-ops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
