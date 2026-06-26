---
trigger: always_on
description: This document provides context about the laptop-local-clusters project for AI assistants.
---

# Project Context for Claude

This document provides context about the laptop-local-clusters project for AI assistants.

## Project Overview

**Purpose**: Standardize local Kubernetes cluster testing across multiple laptops (macOS and Linux) with optional centralized GitOps management via homelab ArgoCD.

**Key Technologies**:
- **k3d** (k3s in Docker) - Lightweight local Kubernetes clusters
- **k3s** - Lightweight production Kubernetes distribution
- **Cilium** (optional, eBPF-based CNI) - For ArgoCD-managed clusters
- **Flannel** (default CNI) - For standalone clusters
- **ArgoCD** - GitOps continuous delivery
- **Docker Desktop / Podman Desktop** - Container runtime

## Architecture

### Two Deployment Modes

**1. Standalone Mode**
- Single laptop with independent cluster
- Uses k3s default Flannel CNI (fast, simple)
- Optional ArgoCD installed locally
- Full autonomy, no external dependencies
- **Ultra-fast setup: ~15 seconds**

**2. Multi-Cluster GitOps Mode**
- Multiple laptop clusters managed by centralized homelab ArgoCD
- Core applications (Cilium with ingress, monitoring, cert-manager) synced from Git
- Testing applications deployed manually per laptop
- Consistent infrastructure across all laptops
- **ArgoCD installs Cilium CNI** (Flannel disabled)

### Multi-Cluster Architecture Flow
```
Homelab Cluster (ArgoCD)
    ↓ (via VPN/Tailscale)
Git Repository (core-apps)
    ↓
Laptop Clusters (labeled: environment=laptop)
    ├─ Core Apps (ArgoCD-managed, synced)
    └─ Testing Apps (manual kubectl/helm)
```

## Directory Structure

```
.
├── README.md                         # Quick start guide
├── CLUSTER_PLAN.md                   # Detailed feasibility analysis
├── MULTI_CLUSTER_SETUP.md            # Multi-cluster setup guide
├── CLAUDE.md                         # This file
├── create-cluster-k3d.sh             # k3d cluster creation (PRIMARY SCRIPT)
├── create-cluster.sh                 # Legacy Talos script (deprecated)
├── install-argocd.sh                 # Install ArgoCD locally (standalone mode)
├── cilium-patch.yaml                 # Legacy Talos config (not used with k3d)
├── scripts/
│   ├── README.md                     # Scripts documentation
│   └── register-cluster.sh           # Register laptop with homelab ArgoCD
├── argocd-apps/
│   ├── README.md                     # ArgoCD apps documentation
│   ├── bootstrap-laptop-management.yaml      # Bootstrap app (apply once to homelab)
│   ├── laptop-clusters-applicationset.yaml   # ApplicationSets (managed by bootstrap)
│   ├── hello-world-app.yaml          # Example ArgoCD app
│   └── core-apps/                    # Core infrastructure apps
│       ├── README.md                 # Core apps documentation
│       ├── cilium/                   # Cilium CNI + Ingress Controller
│       ├── monitoring/               # Prometheus/Grafana
│       └── dev-tools/                # cert-manager
└── test-apps/
    └── hello-world/                  # Sample testing application
```

## Key Concepts

### k3d and k3s
- **k3d**: Tool to run k3s clusters in Docker containers
- **k3s**: Lightweight Kubernetes (CNCF-certified distribution)
- Production-ready but optimized for resource-constrained environments
- Perfect for local development and testing
- Fast startup, low memory footprint

### Container Runtime Detection
- Scripts automatically detect Docker Desktop or Podman Desktop
- Sets `DOCKER_HOST` environment variable if Podman is detected
- Works seamlessly with both runtimes

### ApplicationSet Pattern
- ArgoCD feature for managing multiple clusters
- Uses cluster labels for targeting (`environment=laptop`)
- Single ApplicationSet → Multiple Applications (one per cluster)
- Enables consistent core infrastructure across all laptops

### Sync Waves
- Control deployment order via annotations
- Wave -1: Cilium (must deploy first for networking and ingress)
- Wave 0: Infrastructure (cert-manager)
- Wave 1: Monitoring
- Wave 2+: Applications

### Hybrid Deployment Model
- **Core Apps**: Managed by ArgoCD, always in sync, cannot be modified manually
- **Testing Apps**: Deployed via kubectl/helm, independent per laptop, not in Git

### CNI Selection
- **Standalone mode**: Uses Flannel (k3s default) - fast, simple, battle-tested
- **ArgoCD-managed mode**: Uses Cilium (installed by ArgoCD) - advanced features, observability

## Common Workflows

### Create Standalone Cluster
```bash
./create-cluster-k3d.sh my-cluster
./install-argocd.sh  # optional
```

### Create ArgoCD-Managed Cluster
```bash
# ONE-TIME: Apply bootstrap app to homelab (if not already done)
kubectl apply -f argocd-apps/bootstrap-laptop-management.yaml --context homelab

# Create laptop cluster and register with homelab
./create-cluster-k3d.sh --skip-cilium --register-with-homelab homelab my-laptop
# ArgoCD automatically deploys core apps via ApplicationSets
```

### Deploy Testing Application
```bash
kubectl config use-context k3d-my-cluster
kubectl apply -f test-apps/hello-world/deployment.yaml
```

### Add New Core Application
1. Create `argocd-apps/core-apps/new-app/` with manifests
2. Add ApplicationSet to `laptop-clusters-applicationset.yaml`
3. Commit and push
4. ArgoCD syncs to all registered clusters

### Destroy Cluster
```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nixc0/laptop-local-clusters](https://github.com/nixc0/laptop-local-clusters) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
