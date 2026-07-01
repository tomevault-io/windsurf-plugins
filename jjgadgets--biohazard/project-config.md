---
trigger: always_on
description: Biohazard is a homelab monorepo managing a single production Kubernetes cluster powered by Talos Linux, Cilium (CNI), Rook-Ceph (storage), Flux (GitOps), and Renovate (dependency updates). The goal is maximum automation with minimal manual intervention.
---

# AGENTS.md

## Project Overview

Biohazard is a homelab monorepo managing a single production Kubernetes cluster powered by Talos Linux, Cilium (CNI), Rook-Ceph (storage), Flux (GitOps), and Renovate (dependency updates). The goal is maximum automation with minimal manual intervention.

This is a personal "production" environment used to explore security solutions and infrastructure patterns in a real-world setting. "Glorifying jank that *works*" is the operating philosophy.

## Architecture

- **Cluster**: Biohazard (production Talos + Kubernetes 1.35.x)
- **GitOps**: Flux syncs desired state from this repo (two sources: GitHub `flux-system` + internal `soft-serve`)
- **Networking**: Cilium (CNI/NetworkPolicy/LoadBalancer), Multus (VLAN passthrough), BIRD (BGP), stunner (TURN/STUN), envoy-gateway + ingress-nginx, cloudflared
- **Storage**: Rook-Ceph (HA), VolSync (backups via Restic/rclone), democratic-csi (local-hostpath), external-snapshotter, snapscheduler, fstrim, csi-addons
- **DNS**: k8s-gateway (internal), external-dns (Cloudflare)
- **TLS**: cert-manager + trust-manager
- **Secrets**: SOPS-encrypted (age + PGP) in repo, runtime secrets via **external-secrets** operator pulling from **1Password** into Kubernetes Secrets
- **Databases**: CloudNativePG (Postgres, default + home clusters), Redis, Litestream, Mosquitto (MQTT)
- **Monitoring**: VictoriaMetrics + kube-prometheus-stack, Grafana, Alertmanager, Karma, node-exporter, smartctl-exporter, intel-gpu-exporter, fluentbit + Vector (log pipeline)
- **Autoscaling**: KEDA (event-driven)
- **Container registry cache**: Spegel
- **Hardware acceleration**: node-feature-discovery, intel-device-plugins, nvidia (for LLM/GPU workloads)
- **VMs**: KubeVirt (`_kubevirt/`), with AD and personal VMs
- **Auth/SSO**: Authentik (fronting ingress-nginx via outpost annotations)
- **Git server**: soft-serve (internal, used as a Flux source for app repos)
- **Config management**: `mise` for tool versions, `go-task` for tasks, `talhelper` for Talos config

## Directory Structure

```
kube/
├── bootstrap/flux/         # Initial Flux install manifests (flux-install-localhost.yaml, svc-metrics.yaml)
├── clusters/biohazard/
│   ├── config/             # ExternalSecret → K8s Secret for cluster vars
│   ├── flux/               # Top-level Flux Kustomization: GitRepository, externalsecret, patches
│   │   ├── flux-repo.yaml  # GitRepository + Kustomization with global patches (labelSelector-driven)
│   │   ├── externalsecret.yaml
│   │   └── kustomization.yaml
│   └── talos/              # talhelper config (talconfig.yaml) + SOPS-encrypted talsecret.yaml + watchdog.yaml
├── deploy/
│   ├── core/               # Essential cluster components (see Core Components below)
│   │   ├── _networking/    # cilium, multus, bird (BIRD currently active)
│   │   ├── db/             # pg, redis, litestream, mosquitto
│   │   ├── dns/            # external-dns, internal/k8s-gateway
│   │   ├── flux-system/    # flux healthcheck
│   │   ├── hardware/       # node-feature-discovery, intel-device-plugins, nvidia
│   │   ├── ingress/        # ingress-nginx, envoy-gateway, cloudflare, secrets-sync, stunner
│   │   ├── monitoring/     # victoria, kps, grafana, alertmanager, karma, fluentbit, vector, exporters, keda
│   │   ├── reloader/       # config-reloader for Secret/ConfigMap change detection
│   │   ├── secrets/        # external-secrets operator
│   │   ├── spegel/         # P2P registry cache
│   │   ├── storage/        # rook-ceph (+ cluster), democratic-csi, volsync, fstrim, snapshots, csi-addons
│   │   └── tls/            # cert-manager, trust-manager
│   ├── apps/               # ~60 user-facing apps (see App Pattern below)
│   └── vm/                 # KubeVirt VMs (_kubevirt, ad, jj)
├── repos/flux/             # HelmRepository and OCIRepository sources (helmpatches applied)
└── templates/test/         # Template for testing
.taskfiles/                 # go-task Taskfile includes (one per domain)
ostree/                     # NixOS/oStree host configurations (outside k8s scope)
dots/                       # Dotfiles (vim, k9s configs, etc.)
```

## Core Components (kube/deploy/core)

Components essential for the cluster to operate:

- **Networking** (`_networking/`): Cilium, Multus, BIRD
- **Storage** (`storage/`): Rook-Ceph (+ cluster), democratic-csi, VolSync, fstrim, external-snapshotter, snapscheduler, csi-addons
- **DNS** (`dns/`): external-dns, k8s-gateway
- **TLS** (`tls/`): cert-manager, trust-manager
- **Ingress** (`ingress/`): ingress-nginx, envoy-gateway, cloudflared tunnel, secrets-sync, stunner
- **Monitoring** (`monitoring/`): VictoriaMetrics, kube-prometheus-stack, Grafana, Alertmanager, Karma, metrics-server, fluentbit, Vector, node-exporter, smartctl-exporter, intel-gpu-exporter, KEDA
- **Databases** (`db/`): CloudNativePG, Redis, Litestream, Mosquitto
- **Hardware** (`hardware/`): node-feature-discovery, intel-device-plugins, nvidia
- **Secrets** (`secrets/`): external-secrets
- **Misc**: Spegel, Reloader, flux-system/healthcheck


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JJGadgets/Biohazard](https://github.com/JJGadgets/Biohazard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
