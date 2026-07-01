---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a multi-environment, multi-cloud infrastructure-as-code repository for a home lab environment using GitOps principles. It manages Terraform configurations for cloud resources, Kubernetes workloads via Flux CD, and on-premises virtualization.

**Related Repositories:**
- [anthr76/snowflake](https://github.com/anthr76/snowflake) - System-level configuration for nodes, routers, and infrastructure components

**Additional Context:**
- See `.github/instructions/.instructions.md` for comprehensive coding standards and guidelines

## Development Environment

This repository uses [Flox](https://flox.dev/) (Nix-based development environments) for consistent tooling. All required tools are automatically available:

```bash
# Enter the development environment (automatically via direnv, or manually)
flox activate

# All tools are now available: kubectl, flux, helm, sops, age, just, etc.
```

The environment is defined in `.flox/env/manifest.toml`. direnv auto-activates it
on `cd` via `.envrc` (`eval "$(flox activate)"`).

**Never manually install tools** - they are managed via the Flox manifest. To add a
tool: `flox install <package>` (or edit `.flox/env/manifest.toml`).

`kubectl-volsync` is a custom Go build (not in the catalog) defined under
`.flox/pkgs/kubectl-volsync/`. Build it once with `flox build kubectl-volsync`; the
manifest's activation hook then adds `./result-kubectl-volsync/bin` to PATH.

## Common Commands

### Task Automation
All automation tasks are defined in the `Justfile`. Use `just` instead of custom scripts:

```bash
# List all available tasks
just --list

# Reconcile all Flux GitRepositories and Kustomizations
just flux-reconcile

# Sync Flux GitRepos, Kustomizations, and HelmReleases
just flux-sync

# Suspend all Flux resources
just flux-suspend

# Resume all Flux resources
just flux-resume

# Install Fedora CoreOS for RPI4 (legacy - being phased out)
just burn-fcos-pi4
```

### Kubernetes Operations

```bash
# Switch cluster context
kubectl config use-context <context-name>

# Manually reconcile a specific Flux resource
flux reconcile source git flux-system
flux reconcile kustomization <name>

# Check Flux status
flux get all -A

# View Flux logs
flux logs --all-namespaces
```

### Terraform Operations

```bash
# Format Terraform files
terraform fmt -recursive

# Plan changes
cd armature/prod/<environment>/<service>
terraform plan

# Apply changes
terraform apply
```

### Secret Management

```bash
# Decrypt a SOPS-encrypted file
sops -d <file>.sops.yaml

# Encrypt a file with SOPS
sops -e <file>.yaml > <file>.sops.yaml

# Edit encrypted file in-place
sops <file>.sops.yaml
```

## Architecture

### Directory Structure

- **`armature/prod/`** - Production infrastructure Terraform configurations
  - `cloud-dns/` - Cloudflare DNS management
  - `gcp/` - Google Cloud Platform resources (bootstrap, projects: kutara, top22) - **Being phased out for Talos**
  - `scr1/` - Primary on-premises site (VMs, FCOS configs, switch configs)
  - `nwk1/` - Secondary network site
  - `b2/` - Backblaze B2 storage buckets
  - `tf-states/` - Terraform state configurations

- **`k8s/`** - Kubernetes manifests
  - `base/` - Base Kustomize configurations (each subdirectory = namespace)
  - `clusters/` - Cluster-specific overlays (qgr1-cluster-0, civo-mgmt-0)
  - `rbac/` - RBAC configurations

- **`docs/`** - Infrastructure documentation

### Kubernetes Namespace Organization

**Critical Rule:** Each directory under `k8s/base/` maps 1:1 to a Kubernetes namespace.

Examples:
- `k8s/base/flux-system/` → `flux-system` namespace
- `k8s/base/home/` → `home` namespace
- `k8s/base/media/` → `media` namespace
- `k8s/base/volsync-system/` → `volsync-system` namespace

Namespaces include: database, default, federation, flux-system, home, infra, kube-system, media, monitoring, networking, rook-ceph, volsync-system, kamaji-system, tailscale, tenant-cluster-qgr1, external-secrets, reloader, descheduler, node-feature-discovery.

### Terraform Organization

- **Remote State:** Managed in Terraform Cloud (organizations: "kutara", "rabbito-home")
- **Provider Versions:** Always pinned in `required_providers` blocks
- **Secrets:** Stored in `*.sops.yaml` files, loaded via data sources
- **Naming Convention:** `{service}-{environment}-{resource_type}`

Standard Terraform files: `main.tf`, `variables.tf`, `providers.tf`, `outputs.tf`, `*.sops.yaml`

### GitOps with Flux

- **Flux Operator:** Used for managing Flux installation (see `k8s/base/flux-system/flux-operator/`)
- **HelmRepositories:** All HelmRepository CRs belong in `k8s/base/flux-system/helm-chart-repositories/`
- **Kustomize:** Used for all configuration management
- **Cluster Overlays:** Located in `k8s/clusters/<cluster-name>/`
- **Secret Management:** Migrating to Bitwarden Secrets Manager (see migration notes below)

## Critical Guidelines

### Dependency Management with Renovate

**All version updates go through Renovate.** Renovate manages:
- Container image tags in HelmReleases and Kubernetes manifests
- Helm chart versions in HelmRelease `spec.chart.spec.version`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anthr76/infra](https://github.com/anthr76/infra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
