---
trigger: always_on
description: > Guidelines for AI coding agents operating in the `slinky-slurm` subdirectory of
---

# AGENTS.md — slinky-slurm

> Guidelines for AI coding agents operating in the `slinky-slurm` subdirectory of
> `awsome-distributed-training`. This is an infrastructure-as-code project for deploying
> Slurm on Amazon SageMaker HyperPod EKS via the Slinky Project (SchedMD).

## Project Overview

This project contains Helm values, Kubernetes manifests, Dockerfiles, Terraform/CloudFormation
parameters, Slurm batch scripts, deployment automation scripts, and documentation. There is
**no application source code** (no Python/Go/TS modules). Sbatch scripts are organized by
workload type under `sbatch/` (e.g., `sbatch/fsdp/`) with hardware-profile prefixes (g5, p5).
Infrastructure config files (`params.json`, `custom.tfvars`) are consolidated at the
project root with `ml.g5.8xlarge` defaults; `deploy.sh` overrides values for any user-specified
instance type via `--instance-type` and `--instance-count`.

Key automation scripts:
- **`deploy.sh`** — Infrastructure deployment via CloudFormation or Terraform;
  supports `--training-plan <name>` for reserved capacity (auto-resolves ARN and AZ);
  CFN path is idempotent (create or update based on stack status)
- **`setup.sh`** — Container image build (CodeBuild/local), SSH keys, Helm values generation
- **`install.sh`** — cert-manager, AWS LB Controller (Pod Identity), subnet tagging,
  FSx PVC, MariaDB, Slurm operator, Slurm cluster Helm installs, NLB config;
  supports `--skip-cert-manager`, `--skip-lb-controller`, `--skip-ebs-csi` for
  pre-installed components and `--cluster-name`/`--vpc-id` for bring-your-own-cluster;
  uses `helm upgrade --install` for idempotent operations
- **`destroy.sh`** — Reverse teardown of all deployed resources (including LB Controller
  Pod Identity + IAM, cert-manager, and FSx PVC); warns when `EKS_CLUSTER_NAME` is
  unset and Pod Identity / addon cleanup is skipped; warns when `AWS_ACCOUNT_ID` is
  unavailable and IAM cleanup is skipped; preserves ECR repository and S3 build context
  bucket with manual cleanup commands printed at end of run
- **`lib/deploy_helpers.sh`** — Extracted testable functions sourced by `deploy.sh` and `setup.sh`
- **`params.json`** — CloudFormation parameters (40 params, g5 defaults)
- **`custom.tfvars`** — Terraform variables (g5 defaults)
- **`slurm-values.yaml.template`** — Consolidated Helm values with shell template variables

## Build / Validate / Test Commands

There is no traditional build system (no Makefile, package.json, or pyproject.toml).

### Docker Image Build

```bash
# Authenticate to ECR (required for DLC base image)
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin 763104351884.dkr.ecr.us-east-1.amazonaws.com

# Build the Slurm compute node image
docker buildx build -t dlc-slurmd:latest -f dlc-slurmd.Dockerfile .
```

### Helm Chart Validation

```bash
# Lint a values file against the upstream chart
helm lint <chart-path> -f slurm-values.yaml
helm template <release-name> <chart-path> -f slurm-values.yaml
```

### YAML Validation

```bash
# Validate Kubernetes manifests
kubectl apply --dry-run=client -f lustre-pvc-slurm.yaml
kubectl apply --dry-run=client -f lustre-storageclass.yaml
```

### Markdown Linting

The repo root has `.markdownlint.jsonc` with these rules:
- MD041 (first-line heading): disabled
- MD013 (line length): 100 chars, code blocks excluded
- MD033 (inline HTML): disabled

```bash
# From repo root
npx markdownlint-cli2 "1.architectures/7.sagemaker-hyperpod-eks/slinky-slurm/**/*.md"
```

### CI Static Analysis (PR workflow)

The GitHub Actions workflow `pr-review-and-slurm-test.yml` runs on PRs to `main`:
- `pylint` and `flake8` on any `.py` files
- `bash -n` syntax checking on `.sh` files
- Secrets scanning via grep patterns

### Tests

Bash scripts are tested using [bats-core](https://github.com/bats-core/bats-core) with
bats-assert and bats-support helper libraries. The repo root `conftest.py` provides legacy
pytest fixtures for Docker-based tests but should not be used as a reference for new tests.

```bash
# One-time setup: install bats-core
brew install bats-core            # macOS
# OR: sudo apt-get install -y bats  # Debian/Ubuntu
# OR: npm install -g bats           # cross-platform

# One-time setup: install bats helper libraries
bash tests/install_bats_libs.sh

# Run all bats tests
bats tests/

# Run a specific test file
bats tests/test_deploy.bats

# Verbose output (show test names)
bats --verbose-run tests/test_deploy.bats
```

Test structure:
- `tests/test_deploy.bats` — 72 unit tests for `deploy.sh` and `lib/deploy_helpers.sh`
- `tests/test_setup.bats` — 13 unit tests for `setup.sh` argument parsing, profile
  resolution, and template substitution
- `tests/test_install.bats` — 49 unit tests for `install.sh` argument parsing, version
  constants, install order, skip flags, existing cluster support, `helm upgrade --install`,
  IAM idempotency, `env_vars.sh` dependency validation, and EBS CSI/gp3 phases
- `tests/test_destroy.bats` — 20 unit tests for `destroy.sh` argument parsing, teardown
  order, IAM cleanup, EKS_CLUSTER_NAME warnings, and CodeBuild TF destroy
- `tests/fixtures/` — Independent copies of `params.json`, `custom.tfvars`, and
  `slurm-values.yaml.template` for test isolation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [awslabs/awsome-distributed-ai](https://github.com/awslabs/awsome-distributed-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
