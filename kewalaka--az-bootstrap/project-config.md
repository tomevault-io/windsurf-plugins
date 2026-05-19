---
trigger: always_on
description: This file provides detailed guidance for AI assistants (and human contributors) working on the `az-bootstrap` PowerShell module. The goal is to ensure future updates are consistent, maintainable, and aligned with the module's design philosophy.
---

# Copilot Instructions for az-bootstrap Module

## Overview

This file provides detailed guidance for AI assistants (and human contributors) working on the `az-bootstrap` PowerShell module. The goal is to ensure future updates are consistent, maintainable, and aligned with the module's design philosophy.

---

## Module Purpose

- **az-bootstrap** automates the setup of Azure infrastructure and GitHub repository environments for Infrastructure-as-Code (IaC) projects.
- It is inspired by `azd up` but is focused on bootstrapping foundational cloud and repository configuration for secure, automated deployments using OIDC and GitHub Actions.
- It provides functionality to create and manage multiple environments (dev, test, prod, etc.) within a project, each with appropriate Azure resources and GitHub environment configurations.

Essentially, it bootstraps both the Azure and GitHub sides needed for a secure, OIDC-based deployment pipeline for a new IaC project, starting from a predefined template, and enables ongoing environment management.

---

## What is the module

The az-bootstrap repository contains a PowerShell module designed to automate the initial setup and ongoing environment management for Infrastructure-as-Code (IaC) projects that use Azure and GitHub. It performs the following main tasks:

- Clones a Template: It takes a GitHub template repository URL (e.g., a starter template for Terraform or Bicep) and creates a new repository from it for your specific project (the "target" repository).
- Provisions Azure Core Infrastructure via Bicep: It deploys an Azure Resource Group and **two** Managed Identities (one for plan, one for apply) within your Azure subscription using a subscription-scoped Bicep template (`environment-infra.bicep`). This template leverages AVM modules.
- Configures GitHub for OIDC: It sets up GitHub Environments (e.g., 'dev-iac-plan', 'dev-iac-apply', 'prod-iac-plan', 'prod-iac-apply'), configures Federated Credentials on the Azure Managed Identities to trust these environments, and sets necessary secrets (like Azure tenant ID, subscription ID, client IDs) in the GitHub environments. This allows GitHub Actions workflows in the target repository to securely authenticate to Azure without needing long-lived secrets.
- Sets up Branch Protection: It configures branch protection rules on the target repository to enforce policies, likely related to the configured environments.
- Assigns RBAC Roles: It grants the created Managed Identity the 'Contributor' and 'RBAC Administrator' roles on the Resource Group, enabling it to deploy and manage resources and permissions within that scope.
- Manages Multiple Environments: It supports adding, configuring, and removing additional environments (dev, test, prod, etc.) after initial setup, with each environment having its own Azure resources and GitHub environments.

---

## Key Features

- Creates a new GitHub repository from a template using `gh repo create --template`.
- Clones the new repository locally for further setup.
- Creates Azure infrastructure (Resource Group, **two** Managed Identities - one for plan, one for apply) using a subscription-scoped Bicep template (`environment-infra.bicep`) which utilizes AVM modules.
- Assigns Contributor and "Role Based Access Control Administrator" roles to the managed identities at the resource group level via Bicep.
- Sets up federated credentials for GitHub environments (plan, apply, etc.) on the appropriate Managed Identities via Bicep.
- Configures GitHub environments, secrets, and branch protection in the new solution repository.
- Supports ongoing environment management (adding/removing environments).
- Separates branch protection from environment-specific configurations.
- Designed for extensibility (future support for Azure DevOps, more policies, etc.).

---

## Design Principles

- **SOLID Principles:**
  - Each function should have a single responsibility.
  - Functions should be open for extension but closed for modification.
  - Use dependency injection (pass parameters, avoid global state).
  - Favor composition over inheritance (compose workflows from small, testable functions).
- **Separation of Concerns:**
  - Public functions orchestrate workflows.
  - Private functions perform atomic tasks (e.g., create RG, assign RBAC, set secret).
- **Idempotency:**
  - Functions should not fail if resources already exist (handle 'already exists' gracefully).
- **Explicit Parameters:**
  - Avoid reliance on environment variables unless explicitly passed or documented.
- **Error Handling:**
  - Use try/catch and meaningful error messages.
  - Fail early and clearly if prerequisites are missing.
- **Testability:**
  - All logic should be covered by Pester tests.
  - Use mocks for external dependencies (az, gh, git).

---

## Folder Structure

- `public/` — Exported functions (main entry points, e.g., `Invoke-AzBootstrap`, `Add-AzBootstrapEnvironment`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kewalaka/az-bootstrap](https://github.com/kewalaka/az-bootstrap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
