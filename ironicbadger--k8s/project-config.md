---
trigger: always_on
description: - This repo uses **Flux** for GitOps, not raw kustomize
---

# Project Memory

## GitOps Workflow

- This repo uses **Flux** for GitOps, not raw kustomize
- Do NOT run `kustomize build` to verify changes - push and let Flux reconcile
- Force reconcile with: `flux reconcile kustomization <name> -n flux-system`

## Directory Structure

```
k8s/
├── clusters/                    # Per-cluster configurations
│   ├── m720q/                   # Bare metal cluster
│   │   ├── talos/               # Talos machine configs
│   │   └── flux-kustomizations/ # Flux Kustomization CRDs
│   └── homelab/                 # Proxmox VM cluster
│       ├── cluster.yaml         # Single source of truth for homelab
│       ├── talos/               # Generated Talos configs
│       ├── terraform/           # Generated Terraform code
│       └── flux-kustomizations/
├── infrastructure/              # Shared Kubernetes addons
│   ├── core/
│   │   ├── base/                # Cluster-agnostic base configs
│   │   └── overlays/            # Cluster-specific overrides
│   │       ├── m720q/
│   │       └── homelab/
│   └── sources/                 # HelmRepository definitions
├── apps/                        # User applications
│   ├── base/                    # Base app configs
│   └── overlays/                # Cluster-specific app configs
├── modules/                     # Reusable Terraform modules
├── scripts/                     # Automation scripts and just modules
├── justfile                     # Command runner
└── .sops.yaml                   # SOPS encryption rules
```

## Clusters

### m720q (Bare Metal)
- **Type**: 3x Lenovo m720q nodes (all control planes)
- **OS**: Talos Linux v1.11.5, Kubernetes v1.32.0
- **Endpoint**: https://10.42.0.101:6443
- **Node IPs**: 10.42.0.101-103
- **Config Source**: `clusters/m720q/talos/talconfig.yaml`
- **Features**: KubePrism (port 7445), Cilium CNI, no kube-proxy
- **Storage**: Rook Ceph across 3 NVMe drives

### homelab (Proxmox VMs)
- **Type**: Virtualized on Proxmox
- **Config Source**: `clusters/homelab/cluster.yaml` (single source of truth)
- **Topology**: 1 control plane + 1 worker
- **Provisioning**: `just px confgen && just px create && just px bootstrap`

## Flux Configuration

### Understanding Flux vs Kustomize
- `kustomization.yaml` (lowercase) = kustomize manifest (config.k8s.io)
- `Kustomization` CRD = Flux reconciliation resource (kustomize.toolkit.fluxcd.io)

### Flux Kustomization Hierarchy (m720q)
```
flux-system (root)
  └── clusters/m720q/
        ├── flux-kustomizations/
        │   ├── sources.yaml           → HelmRepositories
        │   ├── infrastructure-core.yaml → Cilium, cert-manager, Tailscale, Headlamp
        │   ├── api-proxy.yaml         → Tailscale API proxy (depends: infrastructure-core)
        │   ├── infrastructure.yaml    → Monitoring stack (depends: infrastructure-core)
        │   ├── rook-ceph-operator.yaml
        │   ├── rook-ceph-cluster.yaml
        │   └── smokeping.yaml         → depends on rook-ceph-cluster
```

### Key Flux Commands
```bash
flux reconcile kustomization <name> -n flux-system  # Force reconcile
flux get kustomizations -A                          # List all kustomizations
flux get helmreleases -A                            # List all HelmReleases
flux logs --follow                                  # Watch Flux logs
```

## Infrastructure Components

### Networking
- **Cilium**: CNI with L2 LoadBalancer (IP pool: 10.42.0.110-149)
- **Tailscale Operator**: Ingress via Tailscale, API proxy for HA access

### Storage
- **Rook Ceph**: Distributed storage (m720q only)
  - Storage class: `ceph-block`
  - Dashboard available via Headlamp

### Certificates
- **cert-manager**: Auto TLS certificates

### Monitoring (m720q only)
- **kube-prometheus-stack**: Grafana + Prometheus
- **Smokeping**: Network latency monitoring

### Management
- **Headlamp**: Kubernetes dashboard with OIDC (https://idp.ktz.ts.net)

## Overlay Pattern

Base configs in `infrastructure/core/base/` are extended by overlays in `infrastructure/core/overlays/<cluster>/`.

### Example: Adding cluster-specific config
```yaml
# infrastructure/core/overlays/m720q/core/headlamp/helmrelease-patch.yaml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: headlamp
spec:
  values:
    config:
      oidc:
        callbackURL: https://m720q-headlamp.ktz.ts.net/oidc-callback
```

### Overlay kustomization.yaml pattern
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - ../../../../base/headlamp        # Reference base
patches:
  - path: helmrelease-patch.yaml     # Apply cluster-specific patches
```

## Tailscale Tagging

All Tailscale resources (Ingresses, ProxyGroups, operator defaultTags) **must** have:
1. `tag:k8s` - generic tag for all Kubernetes-originated nodes
2. `tag:k8s-<cluster-name>` - cluster-specific tag (e.g., `tag:k8s-m720q`, `tag:k8s-homelab`)

**IMPORTANT**: Never remove existing tags like `tag:k8s-operator` or `tag:k8s-funnel` - these are required by Tailscale. Always ADD the required tags alongside existing ones.

### Ingress Example
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    tailscale.com/tags: "tag:k8s,tag:k8s-m720q"
spec:
  ingressClassName: tailscale
```

### ProxyGroup Example
```yaml

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ironicbadger/k8s](https://github.com/ironicbadger/k8s) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
