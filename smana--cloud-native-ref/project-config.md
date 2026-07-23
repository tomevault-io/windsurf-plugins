---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a comprehensive cloud-native platform reference repository implementing GitOps practices with Kubernetes. The repository demonstrates production-ready configurations for building, managing, and maintaining a secure, scalable cloud-native platform using AWS EKS.

## Infrastructure Architecture

The platform is deployed in three sequential stages:

1. **Network Layer** (`opentofu/network/`): VPC, subnets, Route53, and Tailscale VPN
2. **Security Layer** (`opentofu/openbao/`): OpenBao cluster for secrets management and PKI
3. **Kubernetes Layer** (`opentofu/eks/init/` + `opentofu/eks/configure/`): EKS cluster with Flux, Cilium, and Karpenter

### Key Components

- **OpenTofu**: Infrastructure as Code (Terraform alternative)
- **Terramate**: OpenTofu orchestration and stack management
- **Flux**: GitOps continuous delivery
- **Crossplane**: Infrastructure composition from Kubernetes
- **OpenBao**: Secrets management and private PKI
- **Cilium**: Advanced networking and security with eBPF
- **Gateway API**: Modern ingress and traffic routing
- **VictoriaMetrics**: High-performance observability stack

### EKS Bootstrap Architecture

Two-stage OpenTofu deployment: Stage 1 creates the EKS cluster with temporary CNI, Stage 2 replaces it with Cilium and installs Flux.

**Why two stages?** Helm provider needs cluster endpoint at plan time, so Stage 2 runs after the cluster exists.

**Deploy**: `cd opentofu/eks/init && terramate script run deploy`

**Key Files:**
- `opentofu/config.tm.hcl` - Cilium/Flux versions
- `opentofu/eks/init/main.tf` - EKS module with bootstrap addons
- `opentofu/eks/configure/main.tf` - Cilium and Flux helm_releases
- `opentofu/eks/init/helm_values/cilium.yaml` - Cilium Helm values

**Cilium Prefix Delegation (DISABLED):**
Secondary CIDR (100.64.0.0/16) is disabled due to Cilium bug #43493 causing Gateway API L7 proxy failures on cross-node traffic. When fixed, uncomment `cilium-cni-config.tf` and related settings in `cilium.yaml`.

**Pod Subnet Tagging (IMPORTANT):**
The pod subnets (100.64.x.x) must NOT have the `kubernetes.io/role/cni` tag. VPC-CNI uses this tag to discover subnets during Stage 1 bootstrap, which creates orphan ENIs when Cilium takes over in Stage 2. Only use `cilium.io/pod-subnet=true` for these subnets.

**IAM:** EBS CSI and Crossplane use EKS Pod Identity (`xplane-*` resource scope for Crossplane).

### Self-Hosted LLM Platform (opt-in)

Two independent gates govern the self-hosted LLM platform; both must be released for an end-to-end deploy:

| Layer | Gate | Default | Enable |
|---|---|---|---|
| AWS (S3 Files filesystem + IAM) | `opentofu/llm-platform/` Terramate stack tagged `opt-in`, `$TM_LLM_PLATFORM_ENABLED` env-var guard in `workflows.tm.hcl` | skipped | `TM_LLM_PLATFORM_ENABLED=true terramate -C opentofu/llm-platform script run deploy` |
| Kubernetes (vLLM router, NVIDIA plugin, GPU NodePool, LLM apps, LLM EPI) | `clusters/mycluster-0/llm-platform.yaml` umbrella Flux Kustomization with `spec.suspend: true` | skipped | `flux resume kustomization llm-platform -n flux-system` |

The umbrella Kustomization aggregates 5 children under `clusters/mycluster-0-llm-platform/` (kept a sibling of `clusters/mycluster-0/` to keep `flux-system`'s recursive sync from auto-applying the children and bypassing the umbrella suspend). See `clusters/mycluster-0-llm-platform/README.md` for child manifests + teardown procedure. The default `terramate script run deploy` from `opentofu/` and the default Flux reconciliation both leave the cluster LLM-free.

**Autoscaling design** (composition v0.5.0+, [SPEC-001](docs/specs/0001-llm-platform-prometheus-autoscaling/spec.md)): every model defaults `min=1` with a KEDA `ScaledObject` driven by leading vLLM saturation metrics — `running/max-num-seqs` ratio + `kv_cache_usage_perc`. The legacy KEDA HTTP add-on (proxy in the data path, lagging request-count trigger) is no longer used; AI Gateway routes directly to each vLLM Service.

**Experimental TUI client:** OpenCode (used occasionally; Claude Code stays primary). Setup design lives in the standalone [`Smana/opencode-config`](https://github.com/Smana/opencode-config) repo at `docs/2026-05-05-opencode-migration-design.md`.

## Common Commands

### Terramate / OpenTofu

```bash
terramate script run init       # Initialize all stacks
terramate script run preview    # Preview changes
terramate script run deploy     # Deploy platform
terramate script run drift detect  # Check drift

# EKS deploy (both stages)
cd opentofu/eks/init && terramate script run deploy

# Feature branch testing
TF_VAR_flux_git_ref='refs/heads/my-branch' terramate script run deploy

# Individual stack
cd opentofu/<stack> && tofu plan -var-file=variables.tfvars
```

### EKS Cluster

```bash
aws eks update-kubeconfig --region eu-west-3 --name mycluster-0
flux get all
flux suspend kustomization --all
flux resume kustomization --all
```

### OpenBao

```bash
export VAULT_ADDR=https://bao.priv.cloud.ogenki.io:8200
export VAULT_SKIP_VERIFY=true
bao status
bao auth -method=userpass username=admin
```

## Development Workflow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Smana/cloud-native-ref](https://github.com/Smana/cloud-native-ref) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
