---
trigger: always_on
description: Guidance for AI coding agents (Claude Code, Codex, etc.) working in this repository. `CLAUDE.md` imports this file, so keep all agent guidance here.
---

# AGENTS.md

Guidance for AI coding agents (Claude Code, Codex, etc.) working in this repository. `CLAUDE.md` imports this file, so keep all agent guidance here.

## Overview

Home Kubernetes cluster running on 3x MS-01 (i9-13900H, 96GB RAM) nodes (`m0`, `m1`, `m2`) with Talos Linux. Flux CD watches `kubernetes/` and reconciles all manifests. Infrastructure uses Rook-Ceph for block storage, Cilium for CNI, Envoy Gateway for ingress, Authelia + LLDAP for SSO, and 1Password/External-Secrets for secret management.

## Common Commands

All operations use [go-task](https://taskfile.dev). Run `task` to list all available tasks.

**Flux / Kubernetes:**
```sh
task kubernetes:reconcile          # Force Flux to pull latest changes
task kubernetes:hr:restart         # Restart all failed HelmReleases
task kubernetes:sync-secrets       # Force sync all ExternalSecrets
task kubernetes:cleanse-pods       # Delete pods in Failed/Pending/Completed state
task kubernetes:browse-pvc NS=default CLAIM=<name>   # Mount PVC in temp container
task kubernetes:node-shell NODE=<m0|m1|m2>           # Shell into a node
task kubernetes:nfs-pod NS=default                   # Start pod with NFS mounts
```

**VolSync (backups):**
```sh
task volsync:snapshot APP=<app> NS=default           # Trigger immediate backup
task volsync:restore APP=<app> NS=default PREVIOUS=1 # Restore from snapshot
task volsync:list APP=<app> NS=default               # List available snapshots
task volsync:unlock                                  # Unlock all restic repos
task volsync:state-suspend / volsync:state-resume    # Pause/resume all VolSync sources
```

**Talos:**
```sh
task talos:apply-node NODE=<m0|m1|m2>                # Render + apply machine config to a node
task talos:upgrade-node NODE=<m0|m1|m2> VERSION=<v> # Manually upgrade Talos (normally tuppr does this)
task talos:upgrade-k8s                               # Manually upgrade Kubernetes (normally tuppr does this)
task talos:kubeconfig                                # Regenerate kubeconfig
task talos:reboot-node NODE=<m0|m1|m2>               # Reboot a node
task talos:regen-certs                               # Regenerate Talos admin client certs (machine CA from 1Password)
task talos:reset-node NODE=<node> / talos:reset-cluster / talos:shutdown-cluster  # Destructive
```

**Terraform (Cloudflare):**
```sh
task terraform:cf:init             # Init the Cloudflare workspace
task terraform:cf:plan             # Show planned Cloudflare changes
task terraform:cf:apply            # Apply Cloudflare changes
```

**GitHub / Renovate PRs:**
```sh
task github:pr:list                # List open PRs
task github:pr:merge ID=<n>        # Merge a PR
task github:pr:merge:all SKIP_IDS= # Merge all open PRs (use with care)
```

**Bootstrap (initial cluster only):**
```sh
task bootstrap:talos               # Bootstrap Talos cluster
task bootstrap:apps ROOK_DISK=<model>  # Deploy core apps
```

## Repository Structure

```
kubernetes/
├── flux/
│   ├── cluster/ks.yaml        # Flux entry point: loads meta, then every namespace under apps/
│   └── meta/
│       ├── crds/              # Gateway API and other CRDs
│       └── repositories/      # Helm and OCI repository definitions
├── apps/<namespace>/
│   ├── kustomization.yaml     # Lists each app's ks.yaml in this namespace
│   └── <app>/
│       ├── ks.yaml            # Flux Kustomization (dependsOn, components, postBuild vars)
│       └── app/
│           ├── kustomization.yaml   # Lists resources in this dir
│           ├── helmrelease.yaml     # HelmRelease referencing app-template or a chart
│           └── externalsecret.yaml  # Pulls secrets from 1Password
└── components/
    ├── common/                # Namespace, shared repos, cluster-secrets, alert configs
    ├── volsync/               # Adds VolSync (Restic→R2) backup + PVC to an app
    ├── cnpg/                  # Adds a CloudNative-PG database + backup CronJob + ExternalSecret
    └── dragonfly/             # Adds DragonflyDB (Redis-compat) cluster + NetworkPolicy
talos/
├── controlplane.yaml          # Base Talos machine config (rendered with minijinja-cli + op inject)
├── controlplane/<m0|m1|m2>.yaml  # Per-node config patches
└── schematic.yaml             # Talos image factory schematic
terraform/cloudflare/          # DNS records, Cloudflare tunnels, access policies
bootstrap/                     # helmfile + resources for initial cluster setup
docs/                          # mdBook operations docs (published by the docs workflow)
```

## Architecture Patterns

### Adding a New App

1. Create `kubernetes/apps/<namespace>/<app>/ks.yaml`: a `Kustomization` pointing to `./app`, listing `dependsOn`, `components`, and `postBuild.substitute` vars.
2. Create `kubernetes/apps/<namespace>/<app>/app/` with `kustomization.yaml`, `helmrelease.yaml`, and optionally `externalsecret.yaml`.
3. Add `./<app>/ks.yaml` to `kubernetes/apps/<namespace>/kustomization.yaml`. New namespace directories are discovered automatically; there is no top-level `kubernetes/apps/kustomization.yaml`.

### HelmReleases


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mchestr/home-cluster](https://github.com/mchestr/home-cluster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
