---
trigger: always_on
description: This repository contains ### Kubernetes & GitOps
---

# Infrastructure Repository Instructions

This repository contains ### Kubernetes & GitOps
- Follow GitOps principles: all changes via Git commits
- Use Kustomize for configuration management, avoid raw YAML duplication
- **Namespace Organization**: Each directory under `k8s/base/` maps 1:1 to a Kubernetes namespace (e.g., `k8s/base/volsync-system/` → `volsync-system` namespace)
- **Helm Repositories**: All HelmRepository resources belong in `k8s/base/flux-system/helm-chart-repositories/`
- **Secret Management**: Use existing ClusterSecretStore (`gcp-kutara-prod`) via External Secrets, never commit secrets
- Implement proper resource requests/limits for all workloads
- Use NetworkPolicies for security isolation
- Apply consistent labeling: `app.kubernetes.io/name`, `app.kubernetes.io/component`
- Use HelmReleases with versioned charts, avoid `latest` tags
- **VolSync Backup Strategy**: Use VolSync ReplicationSource for PVC backups to rsync.net with proper scheduling
- Implement monitoring for all custom applications
- **Dependency Management**: All container images, Helm chart versions, and dependencies should be managed by Renovate
- Pin specific versions for container images and Helm charts to enable automated updates via Renovate PRsete infrastructure-as-code for a sophisticated home lab environment using GitOps principles. It includes Terraform for infrastructure provisioning, Kubernetes configurations, and Flux for continuous delivery.

## Repository Overview

This is a multi-environment, multi-cloud infrastructure repository that includes:

- **Infrastructure-as-Code (IaC)**: Terraform configurations for cloud resources, DNS, and virtualization
- **GitOps**: Flux CD for Kubernetes application deployment and management
- **Multi-Cloud**: Google Cloud Platform (GCP), Cloudflare, Backblaze B2, DigitalOcean
- **Virtualization**: Libvirt/QEMU for on-premises VMs running Fedora CoreOS
- **Kubernetes**: Multiple clusters with comprehensive workload management
- **Secret Management**: SOPS for encrypted secrets, External Secrets for Kubernetes

**Related Repositories**:
- **System Configuration**: [anthr76/snowflake](https://github.com/anthr76/snowflake) - Contains system-level configuration for nodes, routers, and infrastructure components## Directory Structure

- `armature/prod/` - Production infrastructure Terraform configurations
  - `cloud-dns/` - Cloudflare DNS management
  - `gcp/` - Google Cloud Platform resources (bootstrap, projects, GKE)
  - `scr1/` - On-premises site resources and VMs
  - `nwk1/` - Network site resources
  - `b2/` - Backblaze B2 storage buckets
- `k8s/` - Kubernetes manifests organized by namespace and cluster
  - `base/` - Base Kustomize configurations
  - `clusters/` - Cluster-specific overlays
- `docs/` - Documentation for the infrastructure setup

## Technology Stack

### Infrastructure & Cloud
- **Terraform**: Infrastructure provisioning and state management
- **Terraform Cloud**: Remote state management (organization: "kutara", "rabbito-home")
- **Google Cloud Platform**: Cloud infrastructure, GKE, Secret Manager
- **Cloudflare**: DNS management and CDN
- **Backblaze B2**: Object storage for backups
- **Fedora CoreOS**: Immutable OS for containers and Kubernetes nodes
- **Libvirt/QEMU**: Virtualization for on-premises infrastructure
- **System Configuration**: Node and router configurations managed in [anthr76/snowflake](https://github.com/anthr76/snowflake)

### Development & Automation
- **Flox**: Nix-based development environment for consistent tooling across contributors
- **Justfile**: Task automation and command standardization (preferred over shell scripts)
- **Renovate**: Automated dependency management for container images, Helm charts, and Terraform modules

### Kubernetes & GitOps
- **Flux CD**: GitOps continuous delivery operator
- **Kustomize**: Kubernetes configuration management
- **Helm**: Package management via HelmReleases
- **SOPS**: Secrets encryption with age/PGP keys
- **External Secrets**: Kubernetes secret synchronization
- **MetalLB**: Load balancer for bare metal Kubernetes
- **Cilium**: Container networking (CNI)
- **Rook Ceph**: Distributed storage
- **VolSync**: Asynchronous PVC replication for backup and disaster recovery

### Applications & Services
- **Home Automation**: Home Assistant, ESPHome, Zigbee2MQTT, Z-Wave JS UI
- **Media Services**: Plex ecosystem, cross-seed, qBittorrent
- **Monitoring**: Prometheus, Grafana, Alert Manager
- **Networking**: Unifi Controller, pfSense
- **Databases**: PostgreSQL (CloudNative-PG), Redis, CouchDB
- **Backup & Replication**: VolSync for PVC backup/restore to external storage (rsync.net)

## Coding Standards & Guidelines

### Terraform
- Use consistent resource naming: `{service}-{environment}-{resource_type}`
- Always specify provider versions in `required_providers` blocks
- Use data sources for external references (e.g., SOPS files)
- Implement proper variable validation and descriptions
- Use `terraform fmt` and follow HashiCorp style guidelines
- Store state remotely in Terraform Cloud
- Use SOPS for sensitive variables (*.sops.yaml files)
- **Module Versioning**: Pin Terraform module versions for Renovate to manage updates
- Avoid using `latest` or branch references for module sources


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anthr76/infra](https://github.com/anthr76/infra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
