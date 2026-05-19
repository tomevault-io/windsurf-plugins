---
trigger: always_on
description: This repository implements an enterprise-grade integration between Microsoft Copilot Studio and Azure AI Search using Terraform infrastructure as code and Azure Developer CLI for deployment.
---

# Copilot Instructions

## Project Overview

This repository implements an enterprise-grade integration between Microsoft Copilot Studio and Azure AI Search using Terraform infrastructure as code and Azure Developer CLI for deployment.

## Security-first, production-ready architecture

Adopt Azure Well-Architected Framework security and reliability pillars as mandatory requirements for all code contributions. When generating or modifying code, prioritize security controls and validate implementation against these principles. Use Azure Best Practices: When generating code for Azure, running terminal commands for Azure, or performing operations related to Azure, invoke your `azure_development-get_best_practices` tool if available

**Identity and access management**
- Generate system-assigned managed identities for all Azure services (already implemented for AI Search service identity)
- Create least-privilege RBAC role assignments with specific scopes (implemented: AI Search → OpenAI, AI Search → Storage)
- Use service principal authentication for AI Search connections in production (configurable via `azure_ai_search_service_principal` variable)
- When `use_service_principal = true`, automatically disable `local_authentication_enabled` and set appropriate `authentication_failure_mode`
- Mark all sensitive Terraform variables and outputs with `sensitive = true` (implemented for client secrets)

**Network security**
- Implement private-by-default networking with VNet injection for Power Platform and deployment scripts
- Configure private endpoints for all PaaS services (implemented: AI Search, OpenAI, Storage)
- Create dedicated subnets with Network Security Groups using default-deny rules (basic NSGs implemented)
- Use Private DNS zones for proper name resolution (implemented for all private endpoints)
- Deploy NAT Gateways for controlled outbound access without public IPs

**Data protection**
- Enable encryption at rest with platform-managed keys (Azure defaults)
- Implement private endpoint networking for data plane access
- Configure diagnostic settings and audit logging for all services

**Multi-region reliability**
- Deploy primary and failover regions with paired Azure regions
- Implement cross-region private endpoint connectivity
- Use zone-redundant storage and compute SKUs where available

**Authentication and secrets**
- Use GitHub OIDC workload identity federation for Azure authentication (configured in workflows)
- Set minimal permissions on workflows: `id-token: write, contents: read`
- Support configurable GitHub runner targets via `ACTIONS_RUNNER_NAME` variable
- Never embed long-lived credentials in workflow files or environment variables

**Development environment**
- Support both local and containerized development workflows
- Pre-configure security scanning tools in dev containers
- Maintain consistent tooling versions across environments

## AZD Template Repo Structure (Terraform)

This project structure is typical for Azure Developer CLI (`azd`) templates using **Terraform** for infrastructure provisioning.

## Directory Structure and Key Files

```plaintext
/
├── .azure/                  # Optional: Local azd environment configuration (e.g., config.json)
├── .devcontainer/           # Dev container configuration for consistent development environment
├── .github/                 # GitHub workflows, templates, and project instructions
│   ├── workflows/           # CI/CD workflows for deployment automation
│   ├── instructions/        # Detailed coding and deployment guidelines
│   └── copilot-instructions.md # This file - project overview and conventions
├── .vscode/                 # VS Code workspace settings
├── azd-hooks/               # Azure Developer CLI hooks for deployment lifecycle
│   └── scripts/hooks/       # Pre/post deployment, provision, and package scripts
├── cicd/                    # CI/CD infrastructure (Terraform state, GitHub runners)
│   ├── *.tf                 # Terraform for remote state and self-hosted runners
│   └── github_runner_*/     # VM and Container Apps runner modules
├── decision-log/            # Architectural Decision Records (ADRs)
├── docs/                    # Comprehensive project documentation
├── infra/                   # Main Terraform IaC code for Azure resources
│   ├── main.tf              # Terraform root module (entry point)
│   ├── main.*.tf            # Feature-specific resource definitions
│   ├── variables.tf         # Input variables used by the module
│   ├── outputs.tf           # Output values for azd service bindings
│   ├── provider.tf          # Provider and backend configuration
│   └── modules/             # Reusable Terraform modules
├── src/                     # Source code for the Power Platform solution and utilities
│   ├── powerplatform/       # Copilot Studio agent and Power Platform assets
│   └── search/              # Azure AI Search configuration and data utilities
├── tests/                   # Automated testing suite
│   ├── AISearch/            # Azure AI Search integration tests
│   └── Copilot/             # Copilot Studio functionality tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure-Samples/Copilot-Studio-with-Azure-AI-Search](https://github.com/Azure-Samples/Copilot-Studio-with-Azure-AI-Search) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
