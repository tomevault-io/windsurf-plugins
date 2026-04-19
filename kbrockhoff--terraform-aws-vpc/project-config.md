---
trigger: always_on
description: * **Simplicity:** One clear purpose per operation.
---

# AI Agent Git Ops – Brockhoff Repos

## Principles

* **Simplicity:** One clear purpose per operation.
* **Composable:** Steps build independently; easy to debug.

## Branch Management

* **Name:** `feature/descriptive-name`, `bugfix/issue-description` (kebab-case, branch from `main`).
* **Stay current:** `make update-branch`; resolve conflicts immediately.

## Commits

* **Pre-commit:** `make pre-commit` if not pushing, `make validate` if pushing.
* **Format:** [Conventional Commits](https://www.conventionalcommits.org/)
  `type(scope): description`
* **Scope:** Atomic, working-state, clear intent (“why”).

## Pull Requests

* **Pre-PR:** `make validate`
* **Template:** Use `.github/pull_request_template.md`

## Quality Gates

* **Automated:** `make validate` (must pass all: format, check, lint, test, docs)
* **Manual review:** Focus on simplicity, decoupling, clarity, and testability

## Troubleshooting

* **Rebase/merge conflicts:** Resolve immediately
* **Test/lint errors:** Fix root cause, don’t mask
* **Reset workspace:**
  `cd $(git rev-parse --show-toplevel) && pwd`
  `git reset --hard origin/main`
  `git clean -fd`
  `make clean`

## AI Agent Commands

* **Validate:** `make validate`
* **Quick check:** `make pre-commit`
* **Info:** `make status`
* **Resource count:** `make resource-count`
* **Cleanup:** `make clean`

**Best Practices:**

* Use single-command Make targets; Fail fast (`make validate`); keep context (`make status`); know rollback (`git reset`, `make clean`).

---

**Tip:** Prefer `make <target>` whenever possible.

---
Source: .ruler/instructions.md
---
# AWS VPC Terraform Module Guide for AI Agents

Terraform module which creates VPC resources on AWS. It takes an opinionated approach on dividing up
and configuring subnets. Resources have standardized tags which enable deployment of resources
into the VPC across multiple environments without having to specify VPC, subnet, or security group ids.

## Components

### VPC
- Support both IPv4 and IPv6
- Include non-routable IPv4 CIDR block (100.64.0.0/10) for EKS pods
- Include non-routable IPv6 CIDR block (fd00::/8) for EKS pods
- Tag with `NetworkTags=standard`

### Subnets
- Be economical with the use if IPv4 private addresses
- Support both IPv4 and IPv6
- Provide the following subnet types:
| ID     | AZs | IP Types        | NetworkTags    |
|--------|-----|-----------------|----------------|
| lb     | 2   | private, public | public         |
| wrkr   | 3   | private         | private        |
| db     | 3   | private         | database       |
| cache  | 2   | private         | cache          |
| pod    | 3   | non-routable    | nonroutable    |

### ACLs
- Always manage the default network ACL and configure to allow all traffic.

### Gateways
- Provide reliability flag for single or multi-zone configurations
- Provide the following gateways:
| ID        | Type     | Optional |
|-----------|----------|----------|
| igw       | internet | true     |
| ngw       | nat

### Endpoints
- Provide the following endpoints:
| ID        | Service Name     | Type     | Optional |
|-----------|------------------|-----------|--------- |
| s3        | s3               | Gateway   | true     |
| dynamodb  | dynamodb         | Gateway   | true     |
| <list>    | <aws-id>         | Interface | N/A      |

### Security Groups

Always managed the default security group and configure it to deny all ingress and egress except for HTTPS egress to the VPC.
- Provide the following security groups:
| ID        | NetworkTags    | Ingress       | Egress       |
|-----------|----------------|----------------------|--------------|
| lb        | public         | HTTPS Internet   | HTTPS private, non-routable |
| db        | database       | DB private, non-routable | N/A |
| cache     | cache          | Cache private, non-routable | N/A            |
| vpc       | vpconly        | VPC                  | VPC |
| ep        | endpoint       | HTTPS private, non-routable   | VPC |

### Flow Logs
- Enable flow logs with input variable flag

---
Source: .ruler/terraform-module.md
---
# Terraform Module Guide for AI Agents

## Core Principles

* **One Responsibility:** Each module should have a single, clear purpose.
* **Composition over Configuration:** Build focused modules that work together, not all-in-one solutions.
* **Minimal, Predictable Interfaces:** Expose only essential variables and outputs. Avoid leaking implementation details.
* **Fail Fast:** Validate inputs and stop early on errors.
* **Clear Documentation:** Explain purpose, usage, and design decisions.

## Module Design

**Good Examples:**

```hcl
# Single-purpose modules
module "vpc" { source = "./modules/vpc" name = var.name cidr_block = var.cidr }
module "db" { source = "./modules/rds" name = "${var.name}-db" subnets = module.vpc.db_subnets }
```

**Avoid:**

```hcl
module "monolith" { create_vpc = true; create_db = true; create_lb = true; ... }
```

## Interfaces

* **Inputs:** Only what's needed; group related config in objects; use sensible defaults and validation.
* **Feature Flags:** Use booleans for optional resources.
* **Outputs:** Only what's useful for composition—IDs, endpoints, etc.
  *Don’t expose internal resource details.*

## Patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kbrockhoff/terraform-aws-vpc](https://github.com/kbrockhoff/terraform-aws-vpc) — distributed by [TomeVault](https://tomevault.io/claim/kbrockhoff).
<!-- tomevault:4.0:windsurf_rules:2026-04-18 -->
