---
trigger: always_on
description: Master context document for AI coding agents contributing to this repository.
---

# Agent Guidelines for tfutils/tfscaffold

Master context document for AI coding agents contributing to this repository.
Every agent MUST read this file before starting work.

---

## Project Overview

tfscaffold is a Terraform wrapper framework for controlling multi-environment,
multi-component, multi-account AWS infrastructure. It handles remote state
management, variable file layering, bootstrap bucket creation, and component
execution orchestration.

- **Language:** Bash (main script: `bin/terraform.sh`, ~960 LOC)
- **Default branch:** `master`
- **Current release:** v2.3.3 (April 2026)
- **Maintainer:** Mike Peachey / Jaz (@Zordrak)
- **Minimal dependencies:** bash, AWS CLI, terraform, GNU getopt
- **License:** MIT

## Repository Structure

```
bin/
  terraform.sh        The tfscaffold script — the core of the project
  docs.sh             Recursive terraform-docs generation helper
bootstrap/            Bootstrap Terraform code for creating the S3 state bucket
                      and DynamoDB lock table
components/           Terraform "components" — root modules run directly
  example/            Example component demonstrating conventions
etc/                  Environment-specific terraform variable files:
                      env_{region}_{environment}.tfvars
                      versions_{region}_{environment}.tfvars
                      global.tfvars, {region}.tfvars, group_{group}.tfvars
lib/                  Optional libraries (e.g. Jenkins pipeline groovy)
modules/
  generic/            Reusable Terraform modules (cognito, iam-role, kms,
                      lambda, s3bucket, sftp, sns, subnets, vpc)
  project/            Project-specific modules (empty by default)
plugin-cache/         Default directory for caching Terraform provider plugins
.github/              CI workflows, agent definitions, issue templates
  agents/             Agent definition files (.agent.md)
  instructions/       Auto-loaded coding standards
  ISSUE_TEMPLATE/     Structured issue forms
docs/
  adr/                Architecture Decision Records
```

---

## Core Principles

### 1. Generated Backend Configuration

tfscaffold generates `backend_tfscaffold.tf` at runtime with the correct S3
backend configuration because Terraform does not support variable interpolation
in backend blocks. This file is cleaned up on exit via `trap`. This is
**intentional** — do NOT attempt to replace it with static configuration.
See [ADR-0002](docs/adr/0002-generated-backend-configuration.md).

### 2. Variable File Layering

Variable files are loaded in a specific precedence order: global → region →
group(s) → environment → versions → remote S3 → `-var` overrides. The `-r`
flag always wins for region. Duplicate variables across files trigger a
warning but are not an error (intentional overrides are a core feature).
See [ADR-0003](docs/adr/0003-variable-file-precedence.md).

### 3. Worktree-First

ALL code changes MUST happen in a git worktree under `.worktrees/`. The main
working tree is shared by all agents and the human — modifying it directly
risks conflicts. `.worktrees/` is gitignored.

```bash
# Create a worktree for your work
git worktree add .worktrees/fix-123 -b fix/123-description master

# Do all work inside the worktree
cd .worktrees/fix-123

# Clean up when done (after PR is created)
cd /path/to/main/tree
git worktree remove .worktrees/fix-123
```

**Exception:** Read-only operations (searching, reading files) may use the
main working tree.

### 4. Quality Over Speed

- Read the diff before pushing
- Check your own work before calling it done
- Fix problems when you find them — do not defer

### 5. Minimal Dependencies

The project deliberately has minimal dependencies: bash, AWS CLI, terraform,
and GNU getopt. Do NOT add new external tools or packages without explicit
approval from the maintainer.

### 6. Auto-Approve is Automatic

tfscaffold **automatically** adds `-auto-approve=true` to all `apply` actions
and `-auto-approve` to `destroy` actions (with legacy `-force` fallback for
Terraform < 0.15). Do NOT pass `-auto-approve` manually — it causes "unknown
option" errors. If custom terraform arguments are needed, use `--` to end
tfscaffold argument parsing first.

---

## Branch and PR Workflow

1. **Never commit directly to `master`.** Always use a feature branch.
2. **Branch naming:** `fix/<short-description>` for bugs,
   `feat/<short-description>` for features, `chore/<short-description>`
   for maintenance. Reference the issue number where applicable
   (e.g. `fix/42-unquoted-variable`).
3. **One logical change per branch/PR.** Do not bundle unrelated fixes.
4. **PRs target `master`.** There is no develop or staging branch.
5. **Merge strategy:** The repo uses merge commits (not squash).
   Do not force-push or rebase shared branches.

## Commit Messages

Freeform — no conventional commits standard is enforced. Be descriptive.
Reference issue numbers where applicable (e.g. `Fix #42: ...`).

Historical examples from CHANGELOG:
- `feat: add multi-group support to terraform.sh`
- `fix: region flag not being authoritative over tfvars files`
- `chore: remove deprecated expected_bucket_owner from s3bucket module`

---

## Claim Protocol

Before starting work on any issue, an agent MUST claim it:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tfutils/tfscaffold](https://github.com/tfutils/tfscaffold) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
