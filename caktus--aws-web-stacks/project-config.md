---
trigger: always_on
description: This repository generates CloudFormation templates for EKS-only deployments.
---

# AGENTS.md - Development Instructions for aws-web-stacks

## Overview

This repository generates CloudFormation templates for EKS-only deployments.
All templates must stay under **51,200 bytes** to fit within CloudFormation's
template size limits when combined with other stacks.

## Prerequisites

### 1. Verify AWS Sandbox Access (FIRST STEP)

Sandbox setup is documented in :doc:`sandbox/README`. In short:

```bash
source ~/.sandbox-aws-env
.venv/bin/aws sts get-caller-identity
```

Expected output: account `148142827518`, role `SandboxAdminRole`, region `us-east-1`.

If this fails, stop and ask the user for credentials.

### 2. Python Virtual Environment

The project uses a Python virtual environment at `.venv/`, managed by `uv`. AWS CLI
is located at `.venv/bin/aws` (not in system PATH). Dependencies are defined in
`pyproject.toml`.

```bash
# Install or update dependencies
uv sync
```

To add or remove a dependency, edit `pyproject.toml` and run `uv sync` again.

## Building Templates

### Quick Build

```bash
USE_EKS=on .venv/bin/python -c 'import stack' > content/eks-no-nat.yaml
USE_EKS=on USE_NAT_GATEWAY=on .venv/bin/python -c 'import stack' > content/eks-nat.yaml
USE_EKS=on USE_GOVCLOUD=on .venv/bin/python -c 'import stack' > content/gc-no-nat.yaml
USE_EKS=on USE_GOVCLOUD=on USE_NAT_GATEWAY=on .venv/bin/python -c 'import stack' > content/gc-nat.yaml
```

### Using Makefile

```bash
make clean
make templates
```

This builds all 4 templates: `eks-no-nat.yaml`, `eks-nat.yaml`, `gc-no-nat.yaml`, `gc-nat.yaml`.

### Size Verification

All templates must be under 51,200 bytes:

```bash
ls -la content/*.yaml
```

## Architecture

### Stack Modules (`stack/`)

| Module | Purpose |
|--------|---------|
| `__init__.py` | Entry point - imports all modules, conditional on `USE_EKS`/`USE_GOVCLOUD`/`USE_NAT_GATEWAY` |
| `common.py` | Shared parameters (encryption, KMS), `arn_prefix` for GovCloud compatibility |
| `template.py` | Base `InterfaceTemplate` class with parameter groups |
| `vpc.py` | VPC, subnets, internet gateway, NAT gateway (conditional) |
| `security_groups.py` | Container security group with EKS cluster tagging |
| `eks.py` | EKS cluster, nodegroup, launch template, EBS CSI + Pod Identity add-ons |
| `containers.py` | EKS node instance type, volume size, IAM role/policies |
| `database.py` | RDS instance (mysql/postgres), optional replica, encryption |
| `cache.py` | Memcached (CacheCluster) and Redis (ReplicationGroup), encryption |
| `assets.py` | S3 buckets (public + private), CloudFront distribution for static assets |
| `domain.py` | Domain name parameters, ACM certificate for app domain |
| `logs.py` | CloudWatch logging policy (uses `arn_prefix` for GovCloud) |
| `repository.py` | ECR repository (uses `arn_prefix` for GovCloud) |
| `tags.py` | Stack name tag applied to all resources |
| `utils.py` | `ParameterWithDefaults` class for default overrides |
| `constants.py` | Shared constants like `dont_create_value = "(none)"` |

### Key Design Decisions

1. **EKS-only**: All other deployment modes (EC2, ECS, Elastic Beanstalk, Dokku) were removed.
2. **GovCloud uses EKS**: No special EC2 handling for GovCloud - it uses the same EKS stack.
3. **No AllowedValues for instance types**: Users specify instance types directly as strings.
   This avoids needing to update templates when new instance types are released.
4. **`arn_prefix` for GovCloud**: All managed policy ARNs use `Join("", [arn_prefix, ":iam::aws:policy/..."])`
   to automatically use `arn:aws-us-gov` in GovCloud regions.
5. **Encryption preserved**: All encryption parameters (AES-256, KMS) are retained across database,
   cache, and assets modules.

### Environment Variables

| Variable | Purpose |
|----------|---------|
| `USE_EKS` | Must be set to `on` for all builds |
| `USE_GOVCLOUD` | Set to `on` for GovCloud templates (changes `arn_prefix`) |
| `USE_NAT_GATEWAY` | Set to `on` to include NAT gateway in private subnets |

## Testing

### Sandbox Deployment

**IMPORTANT: Use changesets for iterative testing.** Dropping and recreating the cluster
each iteration wastes ~15 minutes. Create the stack once, then use `update-stack` with
the rebuilt template for subsequent iterations. Only do a full delete/recreate at the very
end to verify a clean deployment works.

**Use `t3.medium` or larger for sandbox nodes.** The default `t3a.micro` (1 vCPU, 1 GB)
cannot fit the EBS CSI controller, CoreDNS, VPC CNI, kube-proxy, and pod identity agent
simultaneously — you'll get "Too many pods" scheduling failures. Use `t3.medium` (2 vCPU,
4 GB) or larger to avoid this.

For sandbox setup (role deployment, credentials), see ``sandbox/readme.md``.

#### Initial stack creation

```bash
source ~/.sandbox-aws-env
export AWS_DEFAULT_REGION=us-east-1

.venv/bin/aws cloudformation create-stack \
  --stack-name pi-sandbox-test \
  --template-body file://content/eks-no-nat.yaml \
  --parameters \
    ParameterKey=DomainName,ParameterValue=example.com \
    ParameterKey=DomainNameAlternates,ParameterValue= \
    ParameterKey=DatabasePassword,ParameterValue=T3stP4ssw0rd1 \
    ParameterKey=EksClusterName,ParameterValue=pi-sandbox-test \
    ParameterKey=EksClusterVersion,ParameterValue=1.35 \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [caktus/aws-web-stacks](https://github.com/caktus/aws-web-stacks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
