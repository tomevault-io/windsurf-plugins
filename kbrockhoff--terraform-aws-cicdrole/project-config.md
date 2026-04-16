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
# AWS Role for CI/CD Pipeline Terraform Module Guide for AI Agents

Terraform module which an IAM Role on AWS which trusts CI/CD system OIDC provider.

## Components

### IAM Role
- Trusts CI/CD system OIDC provider from a select list
- Has permissions to use Terraform S3 backend

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

* **Consistent Naming & Tagging:** Use predictable names and common tags.
* **Version Pinning:** Lock Terraform and provider versions in `versions.tf`.
* **Centralize Data Sources:** Place all external data lookups in one file and pass results to submodules.

## File Structure

```
/main.tf           # Core logic
/variables.tf      # Inputs
/outputs.tf        # Outputs
/locals.tf         # Computed values
/versions.tf       # Version constraints
/dependencies.tf   # Data sources
/examples/         # Usage demos
/modules/          # Optional submodules
/test/             # Automated tests
```

## AWS Best Practices

* Use `aws_iam_policy_document` instead of inline JSON for policies.
* Default to encryption with customer-managed KMS.
* Create log groups explicitly.

## Avoid

* **God Modules:** Don’t build modules that do everything.
* **Too Many Inputs:** Limit options; use environment-based defaults.
* **Implicit Dependencies:** Accept IDs as inputs, not naming assumptions.
* **Tight Coupling:** Modules shouldn’t assume other modules’ structure.

## Documentation & Testing

* Provide clear examples and rationale in `README.md`.
* Use `terraform-docs` for reference docs.
* Ensure modules are easily testable (`terraform plan` with example configs).

## Release & CI/CD

* Tag releases on `main` with semantic versioning.
* Use GitHub Actions (or similar) to run format, validate, and test steps automatically.

## AI Agent Checklist

* [ ] Single responsibility & clear purpose
* [ ] Minimal, validated inputs and meaningful outputs
* [ ] Predictable naming/tagging
* [ ] Centralized external dependencies
* [ ] Example configs and docs provided
* [ ] Tested for `plan` success and edge cases

---

**Summary:**
Favor small, focused modules with clear interfaces and strong defaults. Compose modules for complex scenarios. Document intent and usage. Avoid monolithic, over-configured designs. **Simplicity and clarity beat feature-completeness.**

---
Source: .ruler/terraform-style.md
---
# Terraform Style Guide for AI Agents


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kbrockhoff) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
