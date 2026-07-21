---
trigger: always_on
description: The **Cloud Game Development Toolkit (CGD Toolkit)** is a collection of Terraform modules, scripts, and configurations for deploying game development infrastructure and tools on AWS. The project enables game studios to deploy production-ready infrastructure for Perforce, Jenkins, Unreal Engine Horde, VDI workstations, and other game development tools.
---

# Cloud Game Development Toolkit - AI Agent Guide

## Project Overview

The **Cloud Game Development Toolkit (CGD Toolkit)** is a collection of Terraform modules, scripts, and configurations for deploying game development infrastructure and tools on AWS. The project enables game studios to deploy production-ready infrastructure for Perforce, Jenkins, Unreal Engine Horde, VDI workstations, and other game development tools.

### Project Structure

```text
cloud-game-development-toolkit/
├── assets/              # Reusable scripts, Packer templates, Ansible playbooks
├── modules/             # Terraform modules for game dev infrastructure
│   ├── jenkins/        # Jenkins CI/CD infrastructure
│   ├── perforce/       # Perforce version control
│   ├── teamcity/       # TeamCity CI/CD infrastructure
│   ├── unity/          # Unity-specific tools
│   ├── unreal/         # Unreal Engine tools (Horde, Cloud DDC)
│   └── vdi/            # Virtual desktop infrastructure
├── samples/            # Complete Terraform configurations
└── docs/               # Documentation source
```

### Key Technologies

- **Terraform**: Infrastructure as Code (IaC) for AWS resource provisioning
- **AWS**: Cloud infrastructure provider
- **Packer**: Machine image building
- **Ansible**: Configuration management
- **Docker**: Container images for services

## Design Philosophy

### 1. Modularity and Flexibility

Modules are designed as building blocks, not complete solutions. Users compose modules to fit their specific needs rather than being forced into opinionated architectures.

**Key Principles:**

- Modules provide infrastructure components, not complete solutions
- Configuration decisions happen in examples, not module internals
- Support multiple deployment patterns through simple variables
- Enable customization without requiring module forking

### 2. Conservative Variable Exposure

Every exposed variable is a commitment to backward compatibility. We start with minimal variables based on known use cases and add more when users request them.

**Guidelines:**

- Start with minimal variables
- Add variables based on user demand (demand-driven)
- Default values should work for 80% of use cases
- Easier to add than remove (breaking changes are painful)

### 3. Security by Default

Game development infrastructure often handles sensitive assets and player data. Security mistakes are costly and hard to fix later.

**Security Patterns:**

- No `0.0.0.0/0` ingress rules in module code (users explicitly define allowed access)
- Private-first architecture with controlled external access
- HTTPS enforcement for internet-facing services
- User-controlled security groups with their own rules

### 4. Readability First

Game development teams often include infrastructure newcomers. Clear, understandable code reduces onboarding time and prevents misconfigurations.

**Code Standards:**

- Prefer explicit over implicit configurations
- Use descriptive variable names that explain purpose
- Self-documenting code over clever abstractions
- Comment complex logic with business context

## Module Design Standards

### Naming Conventions

Use descriptive, purpose-driven resource names:

```hcl
# ✅ GOOD - Descriptive names
resource "aws_lb" "nlb" { }                    # Network Load Balancer
resource "aws_lb" "alb" { }                    # Application Load Balancer
resource "aws_security_group" "internal" { }   # Internal communication

# ❌ BAD - Generic names
resource "aws_lb" "this" { }
resource "aws_lb" "this2" { }
```

### Variable Structure

Use a hybrid approach:

- **Flat variables** for simple, common settings
- **Complex objects** for logical grouping when they provide clear value
- **Submodule alignment** - Complex objects that map directly to submodules

```hcl
# Flat for simple settings
variable "vpc_id" {
  type        = string
  description = "VPC ID for deployment"
}

# Complex objects for logical grouping
variable "load_balancer_config" {
  type = object({
    nlb = object({
      enabled         = optional(bool, true)
      internet_facing = optional(bool, true)
      subnets        = list(string)
    })
  })
}
```

### Resource Patterns

**Prefer direct resources over remote modules:**

```hcl
# ✅ PREFERRED - Direct resource creation
resource "aws_eks_cluster" "main" {
  name     = local.cluster_name
  role_arn = aws_iam_role.cluster.arn
  # Direct configuration gives full control
}

# ❌ AVOID - Remote module dependency
module "eks" {
  source = "registry.terraform.io/example/eks/aws"
  # Adds complexity, version dependencies, limited customization
}
```

**When remote modules are needed, fork them first** for full control over changes and updates.

## Testing Strategy

### Unit Tests with Mocked Providers

All modules use Terraform's native test framework with **mocked providers** to validate module logic without creating actual AWS resources.

**Benefits:**

- Zero AWS costs (no resources created)
- No cleanup required

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws-games/cloud-game-development-toolkit](https://github.com/aws-games/cloud-game-development-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
