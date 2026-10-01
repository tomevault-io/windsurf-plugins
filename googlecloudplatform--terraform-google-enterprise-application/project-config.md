---
trigger: always_on
description: This guide is designed for **AI coding assistants, LLMs, and autonomous agents** contributing to the `terraform-google-enterprise-application` repository. It outlines the architectural mental model, directory structure, development workflows, security guardrails, and verification procedures required to generate high-quality, production-ready contributions.
---

# AGENTS.md - AI Agent & LLM Contribution Guide

This guide is designed for **AI coding assistants, LLMs, and autonomous agents** contributing to the `terraform-google-enterprise-application` repository. It outlines the architectural mental model, directory structure, development workflows, security guardrails, and verification procedures required to generate high-quality, production-ready contributions.

---

## 1. Repository Identity & Mission

This repository implements the Google Cloud Platform (GCP) **Enterprise Application blueprint** (`terraform-google-enterprise-application`). It provides an opinionated, production-ready, and secure internal developer platform (IDP) on Google Cloud.

The blueprint extends the foundational security practices of the [Enterprise Foundation blueprint](https://cloud.google.com/architecture/security-foundations) (`terraform-example-foundation`).

### Core Objectives
*   **Automate Multi-Tenant Platform Provisioning**: Provide isolated environments across development, non-production, and production.
*   **Decouple Infrastructure and Application Lifecycles**: Strict separation of duties between Cloud Platform teams and Application teams.
*   **Enforce Security Guardrails by Default**: Zero-trust networking, Private Service Connect (PSC), VPC Service Controls (VPC-SC), OPA/Gatekeeper Policy Controller, and Binary Authorization.
*   **Support Diverse Workloads**: Microservices, GenAI / LLM agents, High-Performance Computing (HPC), High-Throughput Computing (HTC), and multi-cluster topologies.

---

## 2. Architecture & Stage Execution Model

When reasoning about this codebase, agents must maintain a clear mental model of the progressive 6-stage lifecycle:

```
[0-bootstrap / Test Setup] ──> [1/2-multitenant GKE] ──> [3-fleetscope] ──> [4-appfactory] ──> [5-appinfra] ──> [6-appsource]
```

| Stage | Path / Module | Responsibility & Scope | Target Audience |
| :--- | :--- | :--- | :--- |
| **0 - Bootstrap / Test Harness** | `test/setup/`<br>`modules/standalone-harness` | Foundational test projects, Hub VPC networking with Network Connectivity Center (NCC) / VPC Peering, KMS keys, logging buckets, and test Service Accounts. | Platform Admin |
| **2 - Multi-tenant Infrastructure** | `modules/gke`<br>`modules/cluster_network` | Multi-tenant private GKE clusters, Pod/Service CIDR subnets, Cloud NAT, Cloud Armor security policies, and external static IPs. | Platform Admin |
| **3 - Fleet & Governance** | `modules/fleetscope` | GKE Fleet membership, team scopes, namespaces, Config Sync (ACM GitOps), Anthos Service Mesh (ASM), Policy Controller (Gatekeeper constraints), and Kueue. | Platform Admin |
| **4 - App Factory** | `modules/secure-cicd-pipeline` | Tenant admin projects, Cloud Build private worker pools, Artifact Registry repos, Cloud Deploy delivery pipelines, and IAM service accounts. | Platform Admin / Team Lead |
| **5 - App Infrastructure** | `5-appinfra/`<br>`modules/alloydb-psc-setup` | Workload backing resources: AlloyDB/Cloud SQL databases via PSC, Spanner, GCS buckets, Secret Manager secrets, and Kubernetes Workload Identity bindings. | Application Team |
| **6 - App Source & Delivery** | `6-appsource/`<br>`modules/deployment-pipeline` | Application source code, `Dockerfile`, Skaffold configurations, and Kubernetes manifests (Kustomize overlays: `development`, `nonproduction`, `production`). | Application Developer |

---

## 3. Directory Layout & Key Lookup

```
terraform-google-enterprise-application/
├── modules/                        # Reusable Terraform modules
│   ├── gke/                        # Multi-tenant private GKE cluster module
│   ├── fleetscope/                 # GKE Fleet, scopes, Config Sync, ASM, Policy Controller
│   ├── secure-cicd-pipeline/       # Dedicated CI/CD with Cloud Build private pools & VPC-SC
│   ├── deployment-pipeline/        # Cloud Deploy pipelines and targets
│   ├── alloydb-psc-setup/          # AlloyDB with Private Service Connect
│   ├── cluster_network/            # VPC, subnets, secondary ranges, NAT, Cloud Router
│   ├── private_workerpool/         # Cloud Build private worker pools
│   ├── private_install_manifest/   # Manifest mirroring to internal Artifact Registry
│   ├── binary-authz-build-image/   # Binary Authorization attestation builder
│   ├── standalone-harness/         # Single-project / standalone testing harness
│   ├── nat/                        # Cloud NAT router configuration
│   ├── hpc-ai-training-infra/      # HPC AI/ML GPU training infrastructure
│   ├── hpc-monte-carlo-infra/      # HPC Monte Carlo financial batch infrastructure
│   └── htc-infra/                  # HTC batch orchestration with Kueue & Parallelstore
├── examples/                       # End-to-end reference implementations
│   ├── default-example/            # Baseline 3-tier multi-project app deployment (stages 4-6)
│   ├── standalone_single_project_confidential_nodes/ # Single project with Confidential VMs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GoogleCloudPlatform/terraform-google-enterprise-application](https://github.com/GoogleCloudPlatform/terraform-google-enterprise-application) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
