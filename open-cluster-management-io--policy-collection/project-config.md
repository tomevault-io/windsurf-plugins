---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the **policy-collection** repository for Open Cluster Management (OCM). It contains a collection of governance policies that can be deployed to Kubernetes/OpenShift clusters through OCM's policy framework. Policies are used to enforce security, compliance, and configuration standards across managed clusters.

## Repository Structure

Policies are organized in two primary ways:

### By Support Level
- **stable/** - Policies supported by the Open Cluster Management Policy SIG
- **community/** - Policies contributed by the open source community (contributions start here)
- **3rd-party/** - Policies supported by third parties (not by OCM Policy SIG)

### By NIST 800-53 Security Control Families
Within each support level folder, policies are organized by NIST Special Publication 800-53 control families:
- **AC-Access-Control/** - RBAC, authentication, authorization policies
- **CA-Security-Assessment-and-Authorization/** - Compliance operator installation
- **CM-Configuration-Management/** - Operators, deployments, cluster configuration
- **SC-System-and-Communications-Protection/** - Certificates, encryption, security contexts
- **SI-System-and-Information-Integrity/** - Container scanning, runtime security
- **AU-Audit-and-Accountability/** - Logging and audit configurations

### Policy Generator Projects
- **policygenerator/** - Contains Policy Generator examples and PolicySet projects
  - **policy-sets/stable/** - Tested and supported PolicySets (e.g., openshift-plus, acm-hardening)
  - **policy-sets/community/** - Community-contributed PolicySets (e.g., gatekeeper, kyverno, ocp-best-practices)
  - **kustomize/** - Example manifests demonstrating Policy Generator usage

## Policy Architecture

### Policy Anatomy
Policies in this repository follow the OCM policy structure:

```yaml
apiVersion: policy.open-cluster-management.io/v1
kind: Policy
metadata:
  name: policy-name
  annotations:
    policy.open-cluster-management.io/standards: NIST SP 800-53
    policy.open-cluster-management.io/categories: <Control Family>
    policy.open-cluster-management.io/controls: <Specific Control>
spec:
  remediationAction: inform  # or enforce
  disabled: false
  policy-templates:
    - objectDefinition:
        apiVersion: policy.open-cluster-management.io/v1
        kind: ConfigurationPolicy
        # ... policy definition
```

### Policy Distribution
Policies are distributed to managed clusters using four components (all in same namespace):
1. **Policy** - Groups policy templates, smallest deployable unit on hub
2. **Placement** - Selects target managed clusters (replaces deprecated PlacementRule)
3. **PlacementBinding** - Binds Placement to Policy/PolicySet
4. **ManagedClusterSetBinding** - Makes cluster sets available for Placement selection

### Policy Engines
The repository supports multiple policy engines:
- **OCM ConfigurationPolicy** - Native OCM configuration enforcement
- **Gatekeeper** - OPA-based admission control (policies auto-expanded by generator)
- **Kyverno** - Kubernetes-native policy engine (policies auto-expanded by generator)

## Policy Generator

The Policy Generator is a Kustomize plugin that automatically wraps Kubernetes manifests in OCM policies. Key features:
- Converts standard Kubernetes YAML into OCM Policy resources
- Auto-expands Gatekeeper/Kyverno policies with additional compliance detection
- Configured via `policyGenerator.yaml` files in Kustomize directories
- Binary located at: `bin/policy-generator-plugin/PolicyGenerator`

### Policy Generator Workflow
1. Place manifests in a directory (e.g., `policygenerator/kustomize/policy1_deployment/`)
2. Create `policyGenerator.yaml` defining policy name and manifest paths
3. Create `kustomization.yaml` referencing the generator
4. Generate: `kustomize build --enable-alpha-plugins`

## GitOps Deployment

### Using Application Lifecycle (default)
```bash
cd deploy
./deploy.sh -u <git-url> -b <branch> -p <path> -n <namespace>
```

Parameters:
- `-u` - Git repository URL (default: policy-collection main repo)
- `-b` - Branch name (default: main)
- `-p` - Path to policies subdirectory (default: stable)
- `-n` - Target namespace (default: policies)
- `-s` - Sync rate: high/medium/low/off (default: medium)
- `--deploy-app` - Create Application resource for UI visibility

**Important**: You must be a Subscription Administrator. Add yourself to the `open-cluster-management:subscription-admin` ClusterRoleBinding if needed.

### Using ArgoCD
```bash
cd deploy
./argoDeploy.sh -u <git-url> -b <branch> -p <path> -n <namespace>
```

## Validation and Testing

### Validate Policies Locally
Run the validation script to check policy syntax:
```bash
./build/validate-policies.sh
```

This script:
1. Installs kubeconform for YAML validation
2. Downloads OCM CRD schemas for validation
3. Validates all policies in stable/ and community/
4. Installs kustomize and Policy Generator plugin
5. Validates all PolicySet projects by building them

### CI/CD Validation
- GitHub Actions workflow: `.github/workflows/validate.yml`
- Runs on all PRs and pushes to main

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-cluster-management-io/policy-collection](https://github.com/open-cluster-management-io/policy-collection) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
