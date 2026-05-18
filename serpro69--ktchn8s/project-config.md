---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is ktchn8s - a homelab Kubernetes cluster project that uses Infrastructure as Code (IaC) and GitOps to automate provisioning, operating, and updating self-hosted services. The project provisions bare metal servers (via PXE boot), builds a K3s Kubernetes cluster with Cilium CNI, and deploys various applications and services using ArgoCD.

## Key Commands

### Development Environment

```bash
# Enter Nix development shell (REQUIRED for all tooling - provides all necessary tools)
nix develop

# Main deployment command - provisions entire homelab cluster
make ktchn8s
```

### Cluster Management

```bash
# List all metal hosts from ansible inventory
make inventory

# Access Ansible console for manual operations
make console

# Wake up metal servers without re-provisioning (requires correct BIOS config)
make wake

# Run a command on all servers via SSH
make run CMD="your_command"

# Remove a specific node from cluster (drains, deletes, and wipes disk)
make remove NODE_NAME=<node_name>

# Destroy entire cluster and all resources (requires confirmation)
make destroy

# Cleanup ephemeral PXE server resources
make clean
```

### Layer-Specific Commands

```bash
# Provision bare metal layer (PXE boot + OS install + K3s)
make metal

# Deploy system components (ArgoCD, ingress, storage, etc.)
make -C system main

# Deploy external resources (Terraform-managed - Cloudflare, etc.)
make -C external main

# Run post-installation tasks
make finalize

# Serve documentation locally
make docs
```

### Testing & Validation

```bash
# Run metal infrastructure tests (Cilium, LoadBalancer, etc.)
./tests/metal.sh

# Run network verification tests
./tests/network_stage4.sh

# Run tests with specific filter
make -C tests filter=Smoke
```

### Node Management

```bash
# Add nodes: Update metal/inventory/hosts.yml, then run
make metal

# Remove node safely
kubectl drain ${NODE_NAME} --delete-emptydir-data --ignore-daemonsets --force
kubectl delete node ${NODE_NAME}
# Or use: make remove NODE_NAME=${NODE_NAME}

# Wipe specific server disk remotely
make -C metal wipe SERVER=${NODE_NAME} DISK=/dev/nvme0n1
```

## Architecture

The codebase follows a layered architecture with clear separation of concerns:

### Directory Structure

- **`metal/`**: Ansible playbooks for bare metal provisioning
  - PXE boot server setup (ephemeral Docker containers for DHCP/TFTP/HTTP)
  - Fedora Server installation via kickstart
  - K3s cluster creation with Cilium CNI (eBPF-based, kube-proxy replacement)
  - Dynamic inventory from `inventory.sh` reading `metal/inventory/hosts.yml`
  - Kube-vip for control plane HA

- **`system/`**: Core Kubernetes system components
  - ArgoCD for GitOps (self-managed after bootstrap)
  - NGINX ingress controller
  - cert-manager (Let's Encrypt certificates via Cloudflare DNS-01)
  - external-dns (automatic DNS record management)
  - Cloudflared tunnels for secure external access
  - Rook-Ceph for distributed storage
  - Monitoring stack (Loki, Prometheus, Grafana)
  - Kured for automatic node reboots

- **`platform/`**: Service hosting platform components
  - Gitea (self-hosted Git server)
  - Woodpecker CI (continuous integration)
  - Kanidm (identity management/SSO)
  - Container registry
  - Global secrets management (External Secrets Operator)

- **`apps/`**: User-facing applications
  - Homepage (dashboard)
  - Jellyfin (media server)
  - Paperless (document management)
  - Additional apps as needed

- **`external/`**: External resources managed via Terraform
  - Cloudflare DNS zones and API tokens
  - Cloudflare tunnels configuration
  - External secrets (stored in `terraform.tfvars`)

- **`docs/`**: MkDocs documentation
  - Installation guides
  - Architecture documentation
  - How-to guides
  - Troubleshooting

- **`scripts/`**: Utility scripts
  - `backup.py`: Backup management
  - `post-install.py`: Post-installation tasks
  - `helm-diff.py`: Helm chart comparison

### Provisioning Flow

1. **Metal Layer**:
   - Start ephemeral PXE server on controller
   - Wake nodes via WoL
   - PXE boot → Fedora install → K3s cluster with Cilium
   - Configure networking, storage, and control plane

2. **System Layer**:
   - Bootstrap ArgoCD
   - ArgoCD takes over and manages all subsequent deployments
   - Deploy core system components

3. **Platform Layer**:
   - Deploy developer tools and platform services
   - Configure SSO and secrets management

4. **Apps Layer**:
   - Deploy user applications
   - Configure ingress and external access

### Key Technologies

- **Kubernetes**: K3s (lightweight distribution)
- **CNI**: Cilium with eBPF (replaces kube-proxy, provides L2 LoadBalancer)
- **GitOps**: ArgoCD (manages all K8s resources after bootstrap)
- **IaC**: Ansible (metal provisioning), Terraform (external resources)
- **Networking**:
  - Cilium L2 LoadBalancer announcements
  - NGINX ingress controller
  - Cloudflare tunnels for external access
  - Kube-vip for control plane VIP
- **Storage**: Rook-Ceph (distributed storage)
- **CI/CD**: Woodpecker CI with Gitea integration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [serpro69/ktchn8s](https://github.com/serpro69/ktchn8s) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
