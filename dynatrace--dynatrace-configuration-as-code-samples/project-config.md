---
trigger: always_on
description: > **This is the single source of truth for all AI agent instructions in this repository.**
---

# AI Agent Instructions for Dynatrace Configuration-as-Code Samples

> **This is the single source of truth for all AI agent instructions in this repository.**
> All AI tools (GitHub Copilot, Cursor, Claude, Kiro, etc.) should reference this file.

## Repository Overview

This repository contains sample projects demonstrating Dynatrace Configuration as Code using both Terraform and Monaco (Dynatrace-specific CLI tool). The samples cover various use cases including:

- Pipeline observability (GitHub, GitLab, Azure DevOps, ArgoCD)
- Service Level Objectives (SLOs)
- IAM and access control management
- Dashboard creation and management
- Monitoring and alerting configurations
- GRAIL bucket management
- Segment management
- Workflow automation

## Environment Information

- **Dynatrace Environment**: Platform environments (Gen3) using apps.dynatrace.com
- **Authentication**: Platform token or OAuth-based authentication is REQUIRED for Platform environments
- **Primary Tools**: Terraform with Dynatrace provider v1.89+ (recommended), Monaco CLI v2.28.0+
- **Repository Owner**: Dynatrace organization

---

## Core Principles

### 1. Security First

- **NEVER hardcode credentials** - use environment variables exclusively
- **ALWAYS use OAuth authentication or platform tokens** for Dynatrace Platform environments (preferred over API tokens)
- **Token Compatibility**:
  - Classic API tokens can be used together with OAuth or platform tokens
  - Platform tokens and OAuth **cannot** be used together (mutually exclusive)
- **DO NOT use Terraform variables for tokens/credentials** (they get stored in state file - use environment variables or credential vaults)
- When creating new samples or modifying authentication:
  - Use `DYNATRACE_PLATFORM_TOKEN` or OAuth client credentials (`CLIENT_ID`, `CLIENT_SECRET`)
  - Document required OAuth scopes in README files
  - Create `.env.example` files with placeholder values (NEVER real credentials)
  - Use format: `export VAR_NAME="<PLACEHOLDER_DESCRIPTION>"` in shell scripts

### 2. Version Requirements (January 2026)

| Tool               | Minimum Version | Recommended Version | Notes                                    |
|--------------------|-----------------|---------------------|------------------------------------------|
| Monaco             | 2.24.0          | 2.28.0+             | Required for latest Platform features    |
| Terraform          | 1.0.0           | 1.14.0+             | Use latest stable                        |
| Dynatrace Provider | 1.85.0          | 1.89.0+             | Check registry for latest                |
| Dynatrace Platform | -               | Current             | Platform environments required for OAuth |

- Document version requirements in every README
- Check [Terraform Registry](https://registry.terraform.io/providers/dynatrace-oss/dynatrace/latest) for updates
- For detailed information, see [Dynatrace Configuration as Code](https://docs.dynatrace.com/docs/deliver/configuration-as-code)

### 3. Documentation Requirements

Every sample MUST have a README.md with:
- **Title**: Clear description of the use case
- **Use Case Overview**: Business value and technical implementation
- **Prerequisites**: Required Monaco/Terraform version, Dynatrace environment type
- **OAuth Scopes**: Complete list of required scopes (see template below)
- **Environment Variables**: Complete list with descriptions
- **Project/File Structure**: Overview of files and their purpose
- **Setup Instructions**: Step-by-step deployment guide
- **Images/Screenshots**: Expected outcome (where applicable)
- **Verification**: How to verify successful deployment
- **Cleanup Instructions**: How to delete/remove configurations
- **Troubleshooting**: Common issues and solutions
- **Links**: Use official Dynatrace documentation URLs

---

## Terraform Standards

### Provider Configuration

```hcl
terraform {
  required_version = ">= 1.0"

  required_providers {
    dynatrace = {
      version = "~> 1.89"
      source  = "dynatrace-oss/dynatrace"
    }
  }

  backend "local" {
    path = "./terraform.tfstate"
  }
}

provider "dynatrace" {
  # Provider automatically reads from environment variables:
  # - DYNATRACE_ENV_URL for dt_env_url
  # - DT_CLIENT_ID for client_id (OAuth, recommended for Platform)
  # - DT_CLIENT_SECRET for client_secret (OAuth, recommended for Platform)
  # - DT_ACCOUNT_ID for account_id (OAuth, recommended for Platform)
  # OR
  # - DYNATRACE_PLATFORM_TOKEN for dt_platform_token (Platform token, alternative to OAuth)
  # OR
  # - DYNATRACE_API_TOKEN for dt_api_token (legacy, for Classic environments)
  #
  # Note: Platform token and OAuth are mutually exclusive.
  # Do NOT use Terraform variables for credentials as they get stored in state file.
}
```

### Environment Variables for Authentication

**Required Environment Variables:**
```bash
# Environment URL (required)
export DYNATRACE_ENV_URL="https://abc12345.apps.dynatrace.com"

# OAuth credentials (recommended for Platform environments)
export DT_CLIENT_ID="your-client-id"
export DT_CLIENT_SECRET="your-client-secret"
export DT_ACCOUNT_ID="your-account-uuid"

# OR Platform token (alternative to OAuth for Platform environments)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dynatrace/dynatrace-configuration-as-code-samples](https://github.com/Dynatrace/dynatrace-configuration-as-code-samples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
