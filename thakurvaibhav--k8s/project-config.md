---
trigger: always_on
description: This document provides guidance for AI agents working with this Kubernetes platform repository. It outlines repository structure, conventions, common tasks, and important considerations.
---

# AGENTS.md - AI Agent Guide

This document provides guidance for AI agents working with this Kubernetes platform repository. It outlines repository structure, conventions, common tasks, and important considerations.

## Repository Overview

This repository contains Helm charts implementing a "Platform in a Box" - a GitOps-driven Kubernetes platform foundation using Argo CD's App-of-Apps pattern. The platform includes:

- **Traffic Management**: Envoy Gateway, NGINX Ingress Controller, External DNS
- **Observability**: Prometheus/Thanos (metrics), Elastic Stack (logs), Jaeger (traces)
- **Security & Compliance**: Sealed Secrets, Cert-Manager, Kyverno policies
- **Data Services**: Redis
- **GitOps Orchestration**: Argo CD App-of-Apps pattern

## Key Repository Structure

```
k8s/
├── argocd-bootstrap-apps/     # Bootstrap Application manifests for each cluster
│   ├── dev-01.yaml
│   ├── ops-01.yaml
│   ├── prod-01.yaml
│   └── stag-01.yaml
├── charts/                     # Helm charts for platform components
│   ├── app-of-apps/           # Root orchestrator chart
│   ├── cert-manager/
│   ├── envoy-gateway/
│   ├── external-dns/
│   ├── kyverno/
│   ├── logging/               # Elastic Stack (ECK operator)
│   ├── monitoring/            # Prometheus/Thanos
│   ├── nginx-ingress-controller/
│   ├── redis/
│   ├── sealed-secrets/
│   └── jaeger/
├── docs/                      # Detailed architecture and operational guides
│   ├── getting-started.md
│   ├── observability.md
│   ├── traffic-management.md
│   ├── compliance.md
│   ├── argocd-best-practices.md
│   ├── elastic-best-practices.md
│   ├── troubleshooting.md
│   ├── faq.md
│   └── alert-catalog.md
├── scripts/                   # CI/CD and utility scripts
│   ├── scan.sh               # Chart scanning (lint, trivy, checkov)
│   ├── scan-config.yaml      # Scan configuration
│   ├── junit.tpl             # JUnit template for test results
│   └── what-runs-where.sh
└── README.md                  # Main documentation

```

## Critical Conventions

### 1. Branching & Promotion Model

**NEVER modify production values directly.** Follow the promotion flow:

- `dev` branch → `dev-01` cluster (fast iteration)
- `staging` branch → `stag-01` cluster (soak testing)
- `stable` tag → `prod-01` and `ops-01` clusters (immutable production)

**Key Rules:**
- Production always references `stable` tag (immutable)
- Environment-specific values files: `values.<env>.yaml` (e.g., `values.prod-01.yaml`)
- When modifying charts, start in `dev` branch and promote through the pipeline
- Emergency prod fixes: branch off `stable`, fix, retag, then forward-merge to `staging` and `dev`

### 2. Values File Pattern

Each chart has environment-specific values files:
- `values.yaml` - Base/default values
- `values.dev-01.yaml` - Development overrides
- `values.stag-01.yaml` - Staging overrides
- `values.ops-01.yaml` - Operations cluster overrides
- `values.prod-01.yaml` - Production overrides

**When modifying values:**
- Update the appropriate environment file(s)
- Keep base `values.yaml` minimal (common defaults only)
- Environment files should override, not duplicate base values

### 3. Component Enablement

Components are controlled via feature flags in `app-of-apps/values.yaml`:

```yaml
sealedSecrets:
  enable: false  # Toggle component on/off
  # ... other config
```

**Important:** When adding new components or modifying existing ones:
- Update the `app-of-apps` chart to include the new component
- Add enable/disable toggle in base `values.yaml`
- Create environment-specific overrides if needed
- Update the Inventory table in `README.md`

### 4. Sync Waves

Argo CD sync waves control deployment order. Lower numbers deploy first:

```yaml
annotations:
  argocd.argoproj.io/sync-wave: "-5"  # Deploys early (negative = before 0)
  argocd.argoproj.io/sync-wave: "5"   # Deploys later
```

**Dependencies:**
- Sealed Secrets: `-5` (foundational)
- Envoy Gateway: `-2` (needed by other components)
- External DNS: `-3` (needed for DNS automation)
- Most components: `0` (default)
- Applications: `5+` (deploy after platform)

## Common Tasks

### Adding a New Chart

1. **Create chart structure:**
   ```bash
   charts/new-component/
   ├── Chart.yaml
   ├── values.yaml
   ├── values.dev-01.yaml
   ├── values.ops-01.yaml
   ├── values.prod-01.yaml
   ├── values.stag-01.yaml
   ├── templates/
   │   └── [templates]
   └── README.md
   ```

2. **Add to `app-of-apps` chart:**
   - Add component block in `charts/app-of-apps/values.yaml`
   - Add template in `charts/app-of-apps/templates/` (if needed)
   - Include enable toggle and standard fields (project, namespace, source)

3. **Update documentation:**
   - Add entry to Inventory table in `README.md`
   - Document dependencies in Cross-Chart Relationships section
   - Create component-specific README if complex

4. **Test:**
   ```bash
   helm template charts/new-component -f charts/new-component/values.dev-01.yaml
   helm lint charts/new-component
   scripts/scan.sh lint
   ```

### Modifying Existing Charts

1. **Always test locally first:**
   ```bash
   # Template rendering

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Thakurvaibhav/k8s](https://github.com/Thakurvaibhav/k8s) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
