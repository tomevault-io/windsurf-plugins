---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OADP (OpenShift API for Data Protection) is a Kubernetes operator that installs and manages Velero for backup and restore operations in OpenShift clusters. It extends Velero with OpenShift-specific features like Security Context Constraints (SCC), cloud credential management, and monitoring integration.

## Prerequisites

**Go Version**: Go 1.24.0 (with toolchain go1.24.5)

**macOS Users**: Install GNU sed (required for bundle generation and other targets)

```bash
brew install gnu-sed
```

**Container Tool**: Docker or Podman (auto-detected, defaults to Docker if available)

- Override with: `CONTAINER_TOOL=podman make <target>`

**Tool Version Checking**: Run `make versions` to check all tool versions and detect mismatches

## Development Commands

### Essential Commands

```bash
# Discovery and validation
make help                   # Display all available targets with descriptions
make versions               # Check tool versions and detect mismatches

# Development workflow
make test                    # Run unit tests, linting, and validation (recommended before commits)
make build                   # Build manager binary
make deploy-olm             # Deploy for testing via OLM (recommended for PR testing)
make undeploy-olm           # Remove OLM deployment

# Code generation (run after API changes)
make generate               # Generate DeepCopy methods
make manifests              # Generate CRDs and RBAC manifests
make bundle                 # Generate OLM bundle
make api-isupdated          # Check if API is up to date
make bundle-isupdated       # Check if bundle is up to date

# Linting and formatting
make lint                   # Run golangci-lint
make lint-fix               # Fix linting issues automatically
make fmt                    # Format code with go fmt

# Special targets
make update-non-admin-manifests  # Update NAC manifests from external repo
```

### Testing Commands

```bash
make test-e2e               # Run end-to-end tests (requires setup)
make test-e2e-setup         # Setup E2E test environment
make test-e2e-cleanup       # Cleanup after E2E tests

# Test variations
TEST_VIRT=true make test-e2e              # Run virtualization tests (community HCO, KubeVirt 1.8+)
TEST_VIRT_GA=true make test-e2e           # Run virtualization tests (OpenShift Virtualization from redhat-operators)
TEST_UPGRADE=true make test-e2e           # Run upgrade tests
TEST_CLI=true make test-e2e       # Run CLI-based tests

# Run focused tests
GINKGO_ARGS="--ginkgo.focus='test name'" make test-e2e
```

### Cloud Authentication Deployment

Deploy OADP with cloud-native authentication (STS, Workload Identity, WIF):

```bash
make deploy-olm-stsflow         # Deploy with standardized flow UI (interactive)
make deploy-olm-stsflow-aws     # Deploy with AWS STS
make deploy-olm-stsflow-gcp     # Deploy with GCP Workload Identity Federation
make deploy-olm-stsflow-azure   # Deploy with Azure Workload Identity
```

These targets automate cloud credential setup using cloud-native identity providers instead of manual credential files. The standardized flow provides an interactive UI for configuration.

### E2E Test Setup Requirements

E2E tests require these environment variables:

- `OADP_CRED_FILE`: Path to backup location credentials
- `OADP_BUCKET`: S3 bucket name for backups
- `CI_CRED_FILE`: Path to snapshot location credentials
- `VSL_REGION`: Volume snapshot location region
- `BSL_REGION`: Backup storage location region (optional, defaults to us-east-1)

**Test Labels**: Tests are filtered by cloud provider labels: `aws`, `gcp`, `azure`, `ibmcloud`, `virt`, `hcp`, `cli`, `upgrade`

**Common Test Issues**:

- ttl.sh images expire after TTL_DURATION (default 1h), which may cause test failures if running tests long after initial deployment

## Important Environment Variables

**Operator Configuration**:

- `IMG`: Custom operator image (default: `quay.io/konveyor/oadp-operator:latest`)
- `VERSION`: Override version (default: `99.0.0`)
- `OADP_TEST_NAMESPACE`: Namespace for operator (default: `openshift-adp`)

**Image Build and Registry**:

- `CONTAINER_TOOL`: Container tool to use (`docker` or `podman`, auto-detected)
- `TTL_DURATION`: ttl.sh image expiry time (default: `1h`, max: `24h`)
- `BUNDLE_IMG`: Custom bundle image

**Cloud Provider Credentials** (for E2E tests):

- `OADP_CRED_FILE`, `OADP_BUCKET`, `CI_CRED_FILE`: Backup/snapshot credentials
- `VSL_REGION`, `BSL_REGION`: Cloud regions for volume/backup storage locations

## Git Repository Information

**Upstream Repository**: `openshift/oadp-operator`

**IMPORTANT - Pull Request Target**: Always target `oadp-dev` branch for PRs, NOT `main`

**Branch Structure**:

- Development branch: `oadp-dev` (target for all PRs)
- Release branches: `oadp-major.minor` (e.g., `oadp-1.4`, `oadp-1.5`)
- Many remote branches from various contributors exist

You can verify the current default branch with `git ls-remote --symref upstream HEAD`.

## Architecture Overview

### Core APIs (Custom Resources)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openshift/oadp-operator](https://github.com/openshift/oadp-operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
