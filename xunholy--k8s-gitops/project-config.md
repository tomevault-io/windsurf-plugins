---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a Kubernetes GitOps repository for a personal homelab cluster managed with FluxCD and Talos Linux. The cluster follows enterprise-grade security and observability practices, showcasing CNCF ecosystem tools.

## Architecture

- **Operating System**: Talos Linux (minimal, immutable Kubernetes OS)
- **GitOps**: FluxCD with Flux Operator for declarative cluster management
- **Container Runtime**: containerd
- **Networking**: Cilium CNI with Istio service mesh
- **Storage**: Rook-Ceph, OpenEBS, democratic-csi for container-attached storage
- **Monitoring**: Prometheus, Grafana, Loki, Jaeger, Thanos for observability
- **Security**: Kyverno, OPA Gatekeeper for policy management, Falco & Tetragon for runtime security
- **Load Balancing**: MetalLB for bare metal load balancing
- **Chaos Engineering**: Litmus for chaos testing

## Directory Structure

```
├── kubernetes/                       # Kubernetes manifests and configurations
│   ├── apps/
│   │   ├── base/                     # Base application configurations (DRY principle)
│   │   │   └── [system-name]/        # e.g., observability, kube-system, home-system
│   │   │       ├── [app-name]/
│   │   │       │   ├── app/          # HelmRelease, OCIRepository, secrets, values
│   │   │       │   └── ks.yaml       # Flux Kustomization with dependencies
│   │   │       ├── namespace.yaml
│   │   │       └── kustomization.yaml
│   │   └── overlays/
│   │       └── cluster-00/           # Cluster-specific overrides
│   ├── bootstrap/
│   │   └── helmfile.yaml             # Bootstrap Flux Operator and dependencies
│   ├── clusters/
│   │   └── cluster-00/
│   │       ├── flux-system/          # Flux Operator and FluxInstance configs
│   │       ├── secrets/              # Cluster secrets (SOPS encrypted)
│   │       └── ks.yaml               # Root Kustomization
│   ├── components/
│   │   └── common/alerts/            # Shared monitoring alerts
│   └── tenants/                      # Multi-tenant configurations
├── talos/                            # Talos Linux configuration files
│   ├── generated/                    # Generated Talos configs (encrypted)
│   ├── integrations/                 # Cilium, cert-approver integrations
│   └── patches/                      # iSCSI, metrics patches
├── terraform/                        # Infrastructure as Code
│   ├── cloudflare/                   # Cloudflare DNS/CDN configuration
│   └── gcp/                          # GCP KMS, Thanos storage, Velero backups
├── .taskfiles/                       # Task automation definitions
└── docs/                             # Documentation
```

## Common Commands

### Task Management (Primary Build System)
The repository uses [Task](https://taskfile.dev) for automation. All commands should be run via `task`:

```bash
# FluxCD Operations
task flux:bootstrap          # Bootstrap Flux Operator via Helmfile
task flux:secrets           # Install cluster secrets (SOPS decrypt + apply)
task fluxcd:bootstrap       # Alternative bootstrap path
task fluxcd:diff            # Preview FluxCD operator changes

# Talos Operations
task talos:config           # Decrypt and load talosconfig to ~/.talos/config

# Core Operations
task core:gpg               # Import SOPS PGP keys
task core:lint              # Run yamllint

# View available tasks
task --list
```

**Important Variables:**
- `CLUSTER`: cluster-00 (default cluster ID)
- `GITHUB_USER`: xunholy
- `GITHUB_REPO`: k8s-gitops
- `GITHUB_BRANCH`: main

### Pre-commit Hooks
The repository uses pre-commit for code quality:
```bash
pre-commit run --all-files   # Run all pre-commit hooks
```

Active hooks include:
- YAML/JSON/TOML validation
- yamllint (with `.yamllint.yaml` config)
- shellcheck for shell scripts
- Trailing whitespace and EOF fixes

### Secret Management
Secrets are encrypted using [SOPS](https://github.com/mozilla/sops) with dual encryption (PGP + GCP KMS):
```bash
# Edit encrypted files (automatically decrypts/encrypts)
sops path/to/file.enc.yaml

# Decrypt for viewing only
sops -d path/to/file.enc.yaml
```

**SOPS Configuration:**
- **PGP Key**: `0635B8D34037A9453003FB7B93CAA682FF4C9014`
- **Age Key**: `age19gj66fq5v2veu940ftyj4pkw0w5tgxgddlyqnd00pnjzyndevurqx70g4t`
- **GCP KMS**: Used for stored PGP keys
- Encrypted files use `.enc.yaml` or `.enc.age.yaml` suffix

## Key Technologies & Patterns

### GitOps with FluxCD
This repository uses **Flux Operator** instead of traditional `flux bootstrap`:
- **FluxInstance CRDs**: Declaratively manage FluxCD components
- **OCIRepository**: Used for Helm charts instead of HelmRepository (e.g., `oci://ghcr.io/prometheus-community/charts`)
- **Kustomizations**: Define manifest application with SOPS decryption, post-build substitution, and dependency chains
- **HelmReleases**: Reference charts via `chartRef` pointing to OCIRepository
- **Root Kustomization**: Located at `kubernetes/clusters/cluster-00/ks.yaml`

### Application Deployment Pattern
Each application follows this structure:
1. **Base configuration** in `kubernetes/apps/base/[system-name]/[app-name]/`:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xunholy/k8s-gitops](https://github.com/xunholy/k8s-gitops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
