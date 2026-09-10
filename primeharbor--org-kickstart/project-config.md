---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

org-kickstart is a Terraform module for bootstrapping and managing AWS Organizations. It's an opinionated alternative to AWS Control Tower, focusing on essential security and governance features without the complexity and cost. It manages organization structure, security services (GuardDuty, Macie, SecurityHub, Inspector), CloudTrail, AWS Identity Center (SSO), Service Control Policies (SCPs), Resource Control Policies (RCPs), Declarative Policies, and more.

## Architecture

### Core Components

**Management/Payer Account**: The root AWS organization account where org-kickstart is deployed. Manages the organization structure, OUs, and organization-wide settings.

**Security Account**: Special delegated account that holds the CloudTrail bucket, is trusted by all accounts for security-audit role, and serves as delegated administrator for security services (GuardDuty, Macie, SecurityHub, Inspector, SSO, CloudFormation StackSets).

**Four Required OUs**:
- `Governance` - Security and payer accounts
- `Workloads` - Most production accounts
- `Sandbox` - Development accounts with more service freedom
- `Suspended` - Accounts pending closure

### Multi-Provider Architecture

Terraform doesn't handle multi-region well. This module uses pre-defined AWS providers for all default (non-opt-in) regions for both payer and security accounts to enable regional security services. The `scripts/generate_regionblocks.sh` script generates provider configurations dynamically.

**Important**: When adding regional resources, you must:
1. Add provider blocks for each region in both payer and security accounts
2. Use the `generate_regionblocks.sh` script to regenerate provider configurations
3. Run `terraform init` after generating new region blocks

### Module Structure

- `modules/account/` - Manages individual AWS accounts with budgets, contacts, SSO assignments
- `modules/org_policies/` - Generic module for SCPs, RCPs, and Declarative Policies
- `modules/security_services/` - Enables GuardDuty, Macie, SecurityHub, Inspector per region
- `modules/stacksets/` - Deploys CloudFormation StackSets (audit role deployment)
- `modules/billing_alerts/` - SNS-based billing alerts
- `modules/datatrail/` - Advanced CloudTrail with S3 data events

### Key Files

- [main.tf](main.tf) - Provider definitions and data sources
- [organization.tf](organization.tf) - Organization resource and service enablement
- [security_account.tf](security_account.tf) - Security account creation and delegation
- [accounts.tf](accounts.tf) - Module calls for all workload accounts
- [ous.tf](ous.tf) - Organizational unit definitions and OU name-to-ID mapping
- [scps.tf](scps.tf) - Service Control Policies
- [rcps.tf](rcps.tf) - Resource Control Policies
- [declarative_policies.tf](declarative_policies.tf) - EC2 Declarative Policies
- [sso.tf](sso.tf) - AWS Identity Center permission sets and group assignments
- [cloudtrail.tf](cloudtrail.tf) - Organization CloudTrail configuration
- [billing.tf](billing.tf) - CUR reports and billing data bucket
- [audit-role.tf](audit-role.tf) - StackSet for deploying audit role to all accounts

## Terraform Workflow

### Initial Setup for New Organization

See [examples/local-deploy/README.md](examples/local-deploy/README.md) for detailed first-time setup. Key steps:

1. Enable AWS Organizations and SSO manually in console (see BOOTSTRAP.md)
2. Create terraform state bucket and set up backend
3. Run `terraform init`
4. Import organization and payer account using provided scripts
5. Create security account first with targeted apply: `terraform apply -target module.security_account`
6. Disable accounts and SCPs in tfvars for first apply
7. Run `scripts/generate_regionblocks.sh` to create multi-region provider blocks
8. Re-run `terraform init` after generating region blocks
9. Enable additional features incrementally

### Importing Existing Organizations

See [IMPORTING.md](IMPORTING.md) for complete import guidance. The `scripts/import_org.sh` script generates import blocks and tfvars entries for existing resources.

**Important Resources to Import**:
- Organization: `aws_organizations_organization.org`
- Payer account: `aws_organizations_account.payer`
- Security account: `module.security_account.aws_organizations_account.account`
- Existing OUs, SCPs, CloudTrail, SSO resources

### Standard Commands

```bash
# Initialize (run after backend changes or new provider configs)
terraform init -backend-config="<env>.tfbackend"

# Plan changes
terraform plan -var-file="<env>.tfvars" -out=<env>-terraform.tfplan

# Apply changes
terraform apply <env>-terraform.tfplan

# Targeted operations (use sparingly)
terraform plan -var-file="<env>.tfvars" -target <resource>
terraform apply -var-file="<env>.tfvars" -target <resource>

# Show planned changes
terraform show <env>-terraform.tfplan

# Import existing resource
terraform import -var-file="<env>.tfvars" <resource_address> <resource_id>
```

### Common Development Scenarios


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [primeharbor/org-kickstart](https://github.com/primeharbor/org-kickstart) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
