---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the **Scale With Simplicity** repository - DigitalOcean's collection of Reference Architectures (RAs) demonstrating how to build production-grade solutions on DigitalOcean using Terraform. Each RA is a fully deployable example that orchestrates reusable Terraform modules from the [Terraform Module Library](./TERRAFORM-MODULE-LIBRARY.md).

## Repository Structure

```
scale-with-simplicity/
├── modules/                        # Reusable Terraform modules
│   ├── glb-stack/                  # Global Load Balancer stack
│   ├── ipsec-gateway/              # IPSec VPN gateway droplet
│   ├── multi-region-vpc/           # Multi-region VPC mesh with peering
│   └── partner-network-connect-aws/ # DO to AWS via Megaport
├── reference-architectures/
│   └── <ra-slug>/
│       ├── README.md               # Documentation with embedded Mermaid diagram, prerequisites, inputs, outputs
│       ├── Makefile                # Standard targets: lint
│       ├── k8s/                    # Kubernetes manifests (optional, for RAs with K8s resources)
│       │   ├── namespace.yaml
│       │   ├── deployment.yaml
│       │   └── ...
│       └── terraform/              # Main RA implementation (or terraform/1-infra/, 2-cluster/, etc. for multi-stack)
│           ├── main.tf             # Module invocations and RA-specific resources
│           ├── variables.tf        # Input variable definitions
│           ├── outputs.tf          # Outputs for documentation
│           └── terraform.tf        # Backend and provider configuration
├── test/                           # Shared validation scripts
│   └── scripts/                    # Reusable shell scripts for validation
│       ├── terraform-validate.sh   # Runs terraform init, validate, fmt check
│       └── tflint.sh               # Runs tflint with shared config
└── .github/workflows/              # CI workflows for each RA
```

## Common Commands

All reference architectures follow a standard Makefile pattern. Navigate to `reference-architectures/<ra-slug>/` and run:

### Linting and Validation

```bash
# Run all linting (terraform validate, fmt, and tflint)
make lint

# Run just terraform validation
make tf-validate

# Run just tflint
make tflint
```

### Terraform Operations

From within `reference-architectures/<ra-slug>/terraform/`:

```bash
# Initialize terraform
terraform init

# Plan with tfvars file
terraform plan -var-file=<path-to-tfvars>

# Apply deployment
terraform apply -var-file=<path-to-tfvars>

# Destroy resources
terraform destroy -var-file=<path-to-tfvars>
```

**Environment Setup**: Set `DIGITALOCEAN_TOKEN` environment variable with your DigitalOcean API token. Multi-cloud RAs may also require `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`.

## Architecture Principles

### When to Create Modules vs. Inline Resources

**Create a reusable module when**:
- Orchestrating multiple related resources as a cohesive unit
- The pattern would be reused across multiple RAs
- Complex logic needs to be encapsulated with clear inputs/outputs
- Example: multi-region VPC mesh, GLB stack, IPSec gateway

**Use inline resources in main.tf when**:
- Resources are specific to a single RA
- Simple, one-off configurations
- Example: droplets with user_data, certificates, SSH key lookups

### Module References

Reference reusable Terraform modules using relative paths from the `modules/` directory:
```hcl
module "multi_region_vpc" {
  source      = "../../../modules/multi-region-vpc"
  name_prefix = var.name_prefix
  vpcs        = var.vpcs
}
```

The relative path depends on your Terraform directory depth. For reference architectures at `reference-architectures/<ra-slug>/terraform/`, the path is `../../../modules/<module-name>`.

See [TERRAFORM-MODULE-LIBRARY.md](./TERRAFORM-MODULE-LIBRARY.md) for available modules.

### Naming Conventions

- All resources use `name_prefix` variable for consistent naming and tagging
- Droplets and resources are tagged to enable load balancer targeting

## Multi-Stack Deployments

Some RAs split deployment into sequential stacks (e.g., `terraform/1-infra/`, `terraform/2-cluster/`, `terraform/3-environment/`):
- Stack dependencies (e.g., Kubernetes provider needs cluster from previous stack)
- CRD dependencies (e.g., ServiceMonitor requires kube-prometheus-stack)

Each stack has its own terraform directory with separate main.tf, variables.tf, outputs.tf, and terraform.tf files.

### Multi-Stack Makefile Pattern

```makefile
MAKEFILE_PATH := $(abspath $(lastword $(MAKEFILE_LIST)))
MAKEFILE_DIR := $(dir $(MAKEFILE_PATH))
TEST_SCRIPT_DIR := $(realpath $(MAKEFILE_DIR)/../../test/scripts)

.PHONY: tf-validate-infra
tf-validate-infra:
	@cd terraform/1-infra && $(TEST_SCRIPT_DIR)/terraform-validate.sh

.PHONY: tf-validate-cluster
tf-validate-cluster:
	@cd terraform/2-cluster && $(TEST_SCRIPT_DIR)/terraform-validate.sh

.PHONY: tf-validate
tf-validate: tf-validate-infra tf-validate-cluster

.PHONY: tflint
tflint:
	@cd terraform && $(TEST_SCRIPT_DIR)/tflint.sh

.PHONY: lint
lint: tf-validate tflint
```

### Multi-Stack State Management


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [digitalocean/scale-with-simplicity](https://github.com/digitalocean/scale-with-simplicity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
