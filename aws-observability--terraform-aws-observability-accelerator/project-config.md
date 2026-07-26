---
trigger: always_on
description: You are helping a user deploy EKS observability using this Terraform repository.
---

# AGENT.md — AI Agent Instructions for AWS Observability Accelerator

You are helping a user deploy EKS observability using this Terraform repository.
Your job is to guide them through deployment conversationally — gather the info
you need, provision prerequisites, run Terraform, and hand them working dashboard URLs.

## Repository Structure

```
modules/eks-monitoring/          # Core module — all profiles
examples/
  managed-grafana-workspace/     # Prereq: create a Grafana workspace + API token
  eks-cloudwatch-otlp/           # CloudWatch OTLP via EKS add-on (public-ready)
  eks-cloudwatch-container-insights/  # Standalone CW Agent add-on example
  eks-amp-managed/               # AMP with managed collector (agentless)
  eks-amp-otel/                  # AMP with self-managed OTel Collector
dashboards/
  cloudwatch-otlp/               # Container Insights dashboards (CW Agent add-on)
  original/                      # Standard Prometheus dashboards (AMP profiles)
```

## Collector Profiles

| Profile | Collector | Backend | Example | Status |
|---------|-----------|---------|---------|--------|
| `cloudwatch-otlp` | CW Agent EKS add-on | CloudWatch OTLP endpoint | `eks-cloudwatch-otlp/` | Public-ready |
| `managed-metrics` | AMP Managed Scraper (agentless) | AMP | `eks-amp-managed/` | Public-ready |
| `self-managed-amp` | OTel Collector (Helm) | AMP | `eks-amp-otel/` | Public-ready |

### Profile details

**`cloudwatch-otlp`** (recommended for CloudWatch users)
- Deploys the `amazon-cloudwatch-observability` EKS add-on via `aws_eks_addon`
- Add-on includes: CW Agent DaemonSet, Fluent Bit, kube-state-metrics, node-exporter
- Enhanced Container Insights enabled by default
- IAM via EKS Pod Identity — the add-on manages the association inline
- **Prerequisite**: `eks-pod-identity-agent` add-on must be installed on the cluster
- Uses `cloudwatch-otlp/` dashboards (9 dashboards in "CloudWatch Container Insights" folder)
- Dashboards: cluster, containers, gpu-fleet, kubelet, namespace-workloads,
  node-exporter, nodes, unified-service, workloads

**`managed-metrics`** (recommended for AMP users wanting zero management)
- AMP managed scraper — no in-cluster collector to manage
- Requires at least 2 subnets in 2 AZs
- Uses `original/` dashboards with an AMP datasource
- Supports `additional_scrape_jobs` for custom scrape targets

**`self-managed-amp`** (full control)
- OTel Collector with AMP remote write, optional X-Ray traces + CW Logs
- IRSA role with AMP, X-Ray, and CW Logs policies
- Uses `original/` dashboards with an AMP datasource

---

## Quick Start (Demo Scripts)

For a zero-to-demo setup, use the scripts in `demo/`:

| Script | Purpose |
|--------|---------|
| `demo/config.sh` | Set `AWS_REGION` and `CLUSTER_NAME` (defaults: `us-east-1`, `cw-otlp-demo`) |
| `demo/setup.sh` | End-to-end setup: EKS cluster → Grafana → CloudWatch OTLP monitoring → Python demo app |
| `demo/teardown.sh` | Destroys everything in reverse order |

```bash
# 1. Configure (optional — edit region/cluster name)
vi demo/config.sh

# 2. Run
cd demo/
./setup.sh
```

The scripts are re-entrant: re-running skips already-completed steps.

**Prerequisites** (auto-checked by setup.sh): `aws`, `eksctl`, `kubectl`, `terraform`, `docker`

```bash
brew install weaveworks/tap/eksctl kubectl
brew install --cask docker   # then open Docker Desktop once
```

---

## Deployment Playbook

For manual or partial deployments, follow these steps in order. Each step
checks whether the resource exists before creating it.

### Step 0: Gather Information

Ask the user for:

1. **AWS Region** (default: `us-east-1`)
2. **Profile choice** — recommend `cloudwatch-otlp` for CloudWatch, `managed-metrics` for AMP

Then check what already exists:

```bash
# Existing EKS clusters
aws eks list-clusters --region <REGION>

# Existing Grafana workspaces
aws grafana list-workspaces --region <REGION> \
  --query 'workspaces[*].{name:name,id:id,endpoint:endpoint,status:status}'
```

Ask the user:
- Do you have an EKS cluster to use, or should I create one?
- Do you have a Grafana workspace, or should I create one? (optional — dashboards can be skipped)

### Step 1: EKS Cluster (if needed)

If the user has no cluster, create one with eksctl:

```bash
eksctl create cluster \
  --name <NAME> \
  --region <REGION> \
  --version 1.32 \
  --nodegroup-name system \
  --node-type t3.medium \
  --nodes 2 \
  --managed
```

### Step 1b: EKS Pod Identity Agent (required for cloudwatch-otlp)

The `cloudwatch-otlp` profile uses EKS Pod Identity for IAM. Check if the
Pod Identity Agent is installed:

```bash
aws eks list-addons --cluster-name <NAME> --region <REGION>
```

If `eks-pod-identity-agent` is not in the list, install it:

```bash
aws eks create-addon --cluster-name <NAME> --addon-name eks-pod-identity-agent --region <REGION>
```

### Step 2: Grafana Workspace (if needed, optional)

If the user wants dashboards and has no workspace:

```bash
cd examples/managed-grafana-workspace
terraform init
terraform apply -var="aws_region=<REGION>"
```

**Requires**: AWS IAM Identity Center (SSO) configured in the account.

Record the outputs:
```bash
GRAFANA_ENDPOINT=$(terraform output -raw grafana_workspace_endpoint)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws-observability/terraform-aws-observability-accelerator](https://github.com/aws-observability/terraform-aws-observability-accelerator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
