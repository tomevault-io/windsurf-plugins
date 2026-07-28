---
trigger: always_on
description: This file provides context and guidance for GitHub Copilot when working with the Azure Trusted Research Environment (Azure TRE) project.
---

# GitHub Copilot Instructions for Azure TRE

This file provides context and guidance for GitHub Copilot when working with the Azure Trusted Research Environment (Azure TRE) project.

## Project Overview

Azure TRE is an accelerator to assist Microsoft customers and partners who want to build out Trusted Research Environments on Azure. It enables authorized users to deploy and configure secure workspaces and researcher tooling without a dependency on IT teams.

Trusted Research Environments (TREs) enable organizations to provide research and development (R&D) teams secure access to data alongside tooling to ensure productivity while keeping security controls in place.

Core features include:
- Self-service workspace management for TRE administrators
- Self-service provisioning of R&D tooling for R&D teams
- Package and repository mirroring
- Extensible architecture with custom service templates
- Microsoft Entra ID integration
- Airlock for import and export
- Cost reporting

## Primary Technologies

Azure TRE uses the following key technologies:

- **Infrastructure as Code**:
  - Terraform for infrastructure provisioning
  - Porter/CNAB for bundle packaging

- **Languages**:
  - Python (API, resource processor)
  - TypeScript/JavaScript (UI)
  - Bash (deployment scripts)
  - HCL (Terraform)
  - YAML (CI/CD pipelines, Porter manifests)

- **Cloud Services**:
  - Azure services (App Service, Container Registry, Cosmos DB, etc.)
  - Microsoft Entra ID for authentication
  - Azure VMSS for resource processor

- **Development Tools**:
  - Docker for containerization and as devcontainer for Visual Studio Code Dev Containers extension
  - GitHub Actions for CI/CD
  - Make for build/deployment automation
  - TRE CLI for common TRE setup, authentication, and management commands

## Repository Structure

```text
├── .github               - GitHub workflows, issue templates, and configuration
├── cli                   - TRE CLI for common setup, auth, and management commands
├── devops                - DevOps scripts and bootstrapping tools
├── docs                  - Documentation
├── e2e_tests             - pytest-based end-to-end tests
├── api_app               - API source code and docs
├── resource_processor    - VMSS Porter Runner
├── scripts               - Utility scripts
├── ui                    - React-based web UI with TypeScript
└── templates             - Resource templates
    ├── core/terraform    - Terraform definitions of Azure TRE core resources
    ├── shared_services   - Terraform definitions of shared services
    ├── workspace_services - Workspace services
    └── workspaces        - Workspace templates
```

## Coding Conventions

- **General Formatting**:
  - Follow the formatting rules defined in `.editorconfig`
  - Use consistent indentation, line endings, and character encoding as specified
  - Ensure final newlines and trim trailing whitespace per .editorconfig settings

- **Python**:
  - Follow PEP 8 style guidelines
  - Use FastAPI for API endpoints
  - Use pytest for testing

- **Terraform**:
  - Use HCL format
  - Follow module structure conventions
  - Use variables.tf and outputs.tf for module interfaces
  - Include resource tagging for cost tracking

- **TypeScript/JavaScript**:
  - Follow standard ESLint configuration
  - Use Vitest for testing React components
  - Use React Testing Library for component testing
  - Mock FluentUI components in tests due to JSDOM limitations
  - Maintain 80% code coverage across branches, functions, lines, and statements
  - Focus on testing user interactions and component behavior
  - Use semantic queries (getByRole, getByLabelText) over test IDs when possible
  - Use npm for package management

- **YAML**:
  - Use consistent indentation (2 spaces)
  - Follow Porter best practices for bundle manifests

- **Git**:
  - Update CHANGELOG.md for all significant changes
  - Reference issue numbers in commit messages

## Environment Assumptions

- **Azure**: All resources are deployed in Azure
- **Microsoft Entra ID**: Used for authentication and authorization
- **Networking**: Core infrastructure uses hub-spoke networking model
- **Security**: Zero-trust security model with strict network boundaries
- **Deployment**: CI/CD through GitHub Actions

## Template Structure

Azure TRE uses Porter bundles to define workspaces, workspace services, and user resources. These bundles consist of:

### porter.yaml

This is the main Porter manifest file that defines:
- Credentials required for deployment
- Parameters and their defaults
- Actions (install, upgrade, uninstall)
- Mixins used (terraform, exec, etc.)
- Outputs from the deployment

Example structure:
```yaml
name: tre-service-example
version: 0.1.0
description: "An example TRE service"
registry: azuretre
dockerfile: Dockerfile.tmpl

credentials:
  - name: azure_tenant_id
    env: ARM_TENANT_ID
  - name: azure_subscription_id
    env: ARM_SUBSCRIPTION_ID
  - name: azure_client_id
    env: ARM_CLIENT_ID
  - name: azure_client_secret
    env: ARM_CLIENT_SECRET

parameters:
  - name: param_name
    type: string
    description: "param_description"
    # ... additional parameters

mixins:
  # ... mixins used

install:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/AzureTRE](https://github.com/microsoft/AzureTRE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
