---
trigger: always_on
description: This repository is the **reference implementation** for the [Azure Kubernetes Service (AKS) baseline cluster](https://aka.ms/architecture/aks-baseline) architecture from Azure Architecture Center. It demonstrates the **minimum recommended baseline** for most AKS clusters and serves as the starting point for architectural conversations. Adapt it to your specific requirements before production use.
---

# Copilot Instructions for AKS Baseline

This repository is the **reference implementation** for the [Azure Kubernetes Service (AKS) baseline cluster](https://aka.ms/architecture/aks-baseline) architecture from Azure Architecture Center. It demonstrates the **minimum recommended baseline** for most AKS clusters and serves as the starting point for architectural conversations. Adapt it to your specific requirements before production use.

## Tech Stack

### Infrastructure as Code

- **Bicep** - All Azure infrastructure definitions
- **Azure CLI** - Deployment execution via `az deployment group create`

### Azure Services

- **AKS** - Kubernetes cluster with Azure CNI Overlay networking
- **Azure Firewall** (Premium) - Egress traffic control and threat intelligence
- **Azure Application Gateway** (with WAF) - Ingress and TLS termination
- **Azure Key Vault** - Secrets and certificate management
- **Azure Container Registry** - Private container image storage
- **Azure Monitor / Log Analytics** - Observability and diagnostics
- **Microsoft Entra ID** - Identity and RBAC integration

### Kubernetes Components

- **Flux** (AKS-managed extension) - GitOps operator for cluster bootstrapping
- **Gateway API** (AKS App Routing with Istio) - Ingress via Envoy gateway proxy
- **Secrets Store CSI Driver** (AKS add-on) - Key Vault integration
- **Azure Workload Identity** (AKS add-on) - Pod identity management

### Manifests

- **Kustomize** - Workload manifest composition (`workload/kustomization.yaml`)
- **Kubernetes YAML** - Cluster baseline configurations

## Architecture Overview

This is an **infrastructure-focused** implementation using a hub-spoke network topology. It does not focus on workloads—the sample ASP.NET Core app exists only to demonstrate the infrastructure.

### Network Topology

- **Hub VNet** (`network-team/hub-regionA.bicep`): Central connectivity containing Azure Firewall (managed egress), Azure Bastion (secure management access), and a gateway subnet for VPN/ExpressRoute.
- **Spoke VNet** (`network-team/spoke-BU0001A0008.bicep`): Contains the AKS cluster with dedicated subnets for Application Gateway, ingress resources, cluster nodes, Private Link endpoints, and the API server (private cluster).

### Team Separation Model

> **Note:** This repository is written "in world" as a reference implementation for the fictional company "Contoso Bicycle." It assumes organizational separation of duties typical in enterprises—networking, platform, and application teams operating independently. The identifier `BU0001A0008` represents a fictional workload tracking convention (Business Unit 0001, Application 0008) that large organizations use to track workloads across cost centers and other common boundaries.

The directory structure models organizational separation of duties:

| Directory | Team | Purpose |
|-----------|------|---------|
| `network-team/` | Networking | Hub-spoke topology, firewall rules, VNet peering |
| `workload-team/` | Platform/DevOps | AKS cluster, ACR, Key Vault, App Gateway, monitoring |
| `cluster-manifests/` | Platform/DevOps | Kubernetes baseline configs deployed via Flux GitOps |
| `workload/` | Application | Sample workload manifests (Kustomize) |

### Traffic Flow

1. Client → Application Gateway (TLS termination, WAF inspection)
2. App Gateway → Internal Load Balancer (re-encrypted with wildcard cert)
3. Load Balancer → Envoy gateway proxy (TLS termination)
4. Envoy → Workload pods (HTTP)

All egress flows through Azure Firewall with explicit allow rules.

## Deployment

Deployments are executed via Azure CLI with Bicep—follow the numbered docs in `docs/deploy/` sequentially.

```bash
# Deploy hub network
az deployment group create -g rg-enterprise-networking-hubs \
  -f network-team/hub-regionA.bicep \
  -p nodepoolSubnetResourceIds="['$NODEPOOL_SUBNET_RESOURCE_ID']"

# Deploy spoke network
az deployment group create -g rg-enterprise-networking-spokes \
  -f network-team/spoke-BU0001A0008.bicep \
  -p hubVnetResourceId=$HUB_VNET_ID

# Deploy AKS cluster and supporting services
az deployment group create -g rg-bu0001a0008 \
  -f workload-team/cluster-stamp.bicep \
  -p targetVnetResourceId=$SPOKE_VNET_ID \
  -p clusterAdminMicrosoftEntraGroupObjectId=$CLUSTER_ADMIN_GROUP_ID ...
```

## Conventions

### Bicep Files

- Use `@description()` decorators on all parameters
- Parameters: camelCase (e.g., `targetVnetResourceId`)
- Resource names: kebab-case with location suffix for multi-region support (e.g., `vnet-${location}-hub`, `la-hub-${location}`). This convention enables deploying the same templates across different Azure regions.
- Always configure diagnostic settings to send logs to Log Analytics
- Modules live in `modules/` subdirectories when used. This reference implementation intentionally uses minimal modularization to keep templates readable and self-contained. Modularize further for reusability in production deployments.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mspnp/aks-baseline](https://github.com/mspnp/aks-baseline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
