---
trigger: always_on
description: This is a Helm chart repository maintained by **ckotzbauer** that hosts six Kubernetes Helm charts. The charts are published to a GitHub Pages-based Helm repository at `https://ckotzbauer.github.io/helm-charts` and are also listed on [Artifact Hub](https://artifacthub.io/packages/search?repo=ckotzbauer). The repository contains Kubernetes operators, monitoring tools, and infrastructure utilities -- all focused on Kubernetes cluster operations, security, and observability.
---

# CLAUDE.md

## Project Overview

This is a Helm chart repository maintained by **ckotzbauer** that hosts six Kubernetes Helm charts. The charts are published to a GitHub Pages-based Helm repository at `https://ckotzbauer.github.io/helm-charts` and are also listed on [Artifact Hub](https://artifacthub.io/packages/search?repo=ckotzbauer). The repository contains Kubernetes operators, monitoring tools, and infrastructure utilities -- all focused on Kubernetes cluster operations, security, and observability.

## Tech Stack

- **Helm** (v3) -- all charts use Helm templating with Go templates
- **chart-testing (ct)** -- linting and integration testing via `ct lint` and `ct install`
- **chart-releaser** -- automated chart releases via `helm/chart-releaser-action`
- **kind** -- ephemeral Kubernetes clusters for CI integration tests
- **GitHub Actions** -- CI/CD pipeline
- **Renovate + Dependabot** -- automated dependency updates (Renovate for chart image pins, Dependabot for GitHub Actions)

## Project Structure

```
helm-charts/
├── charts/
│   ├── access-manager/        # RBAC operator chart (apiVersion: v1)
│   ├── cadvisor/              # cAdvisor DaemonSet chart (apiVersion: v1)
│   ├── chekr/                 # Cluster inspection utility chart (apiVersion: v2)
│   ├── postgres-operator/     # Zalando Postgres Operator chart (apiVersion: v2)
│   ├── sbom-operator/         # SBOM cataloging operator chart (apiVersion: v2)
│   └── vulnerability-operator/ # Vulnerability scanning operator chart (apiVersion: v2)
├── .github/
│   ├── workflows/
│   │   ├── lint-test.yaml     # PR lint + integration test
│   │   ├── release.yaml       # Chart release on push to main
│   │   ├── stale.yml          # Stale issue management
│   │   ├── label-issues.yml   # Issue/PR label automation
│   │   └── size-label.yml     # PR size labeling
│   ├── dependabot.yml         # GitHub Actions dependency updates
│   └── label-commands.json    # Slash-command label config
├── ct.yaml                    # chart-testing configuration
├── renovate.json              # Renovate config (extends ckotzbauer/renovate-config)
├── LICENSE                    # Apache 2.0
└── README.md
```

Each chart follows a standard structure:

```
charts/<name>/
├── Chart.yaml
├── values.yaml
├── .helmignore
├── README.md
├── templates/
│   ├── _helpers.tpl
│   ├── deployment.yaml (or daemonset.yaml)
│   ├── serviceaccount.yaml
│   ├── clusterrole.yaml
│   ├── clusterrolebinding.yaml
│   ├── NOTES.txt
│   └── ... (chart-specific templates)
├── ci/                        # (some charts) values files for ct install tests
└── crds/                      # (postgres-operator only) CRD definitions
```

## Charts Overview

### 1. access-manager

- **Chart version:** 0.14.1 | **App version:** 0.13.0 | **API version:** v1
- **Purpose:** Kubernetes operator to simplify RBAC configurations. Manages `RbacDefinition` and `SyncSecretDefinition` custom resources (CRDs defined inline in `templates/crd.yaml`).
- **Image:** `ghcr.io/ckotzbauer/access-manager` (pinned by SHA256 digest)
- **Workload:** Single-replica Deployment with optional leader election when `replicas > 1`
- **RBAC:** Binds to `cluster-admin` ClusterRole
- **Templates:** deployment, serviceaccount, clusterrolebinding, crd, NOTES.txt
- **Key values:** `replicas`, `image`, `resources` (defaults: 10m/128Mi request, 50m/128Mi limit), `securityContext` (non-root UID 1001, read-only root filesystem, all capabilities dropped)

### 2. cadvisor

- **Chart version:** 2.4.1 | **App version:** 0.52.1 | **API version:** v1
- **Purpose:** Deploys Google cAdvisor as a DaemonSet for container resource monitoring and metrics export.
- **Image:** `gcr.io/cadvisor/cadvisor:v0.52.1` (pinned by SHA256 digest)
- **Workload:** DaemonSet with host path volume mounts (`/`, `/var/run`, `/sys`, `/var/lib/docker`, `/dev/disk`)
- **Networking:** ClusterIP Service on port 8080; optional `hostNetwork` mode
- **Monitoring:** Optional Prometheus `ServiceMonitor` (gated by `metrics.enabled`)
- **Security:** Optional PodSecurityPolicy, ClusterRole/ClusterRoleBinding (gated by `podSecurityPolicy.create`); optional securityContext for privileged mode
- **Templates:** daemonset, service, serviceaccount, servicemonitor, psp, clusterrole, clusterrolebinding, NOTES.txt
- **Key values:** `container.additionalArgs` (housekeeping intervals, disabled metrics), `container.hostPaths`, `metrics.enabled/interval/scrapeTimeout`, `podSecurityPolicy.create`, `commonLabels`, `podLabels`

### 3. chekr

- **Chart version:** 0.5.1 | **App version:** 0.5.1 | **API version:** v2, type: application
- **Purpose:** Inspection utility for Kubernetes cluster maintenance. Runs scheduled checks (deprecation analysis, resource usage) as CronJobs and serves HTML/JSON reports via an nginx webserver Deployment.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ckotzbauer/helm-charts](https://github.com/ckotzbauer/helm-charts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
