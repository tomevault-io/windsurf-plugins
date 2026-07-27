---
trigger: always_on
description: This is an Azure DevOps extension that provides tasks for installing and executing Terraform commands from Azure Pipelines. The extension is written in TypeScript and includes multiple components that work together to provide Terraform integration.
---

# Copilot Instructions for Azure Pipelines Extension for Terraform

This is an Azure DevOps extension that provides tasks for installing and executing Terraform commands from Azure Pipelines. The extension is written in TypeScript and includes multiple components that work together to provide Terraform integration.

## Project Overview

The extension contains:
- **Terraform CLI Task** (`tasks/terraform-cli`): Executes Terraform CLI commands (init, plan, apply, destroy, etc.)
- **Terraform Installer Task** (`tasks/terraform-installer`): Installs Terraform on build agents
- **Terraform Plan View** (`views/terraform-plan`): Renders Terraform plan outputs in Azure Pipelines UI

## Code Standards

### Required Before Each Commit
- Run tests before committing to ensure changes don't break existing functionality
- For terraform-cli: Navigate to `tasks/terraform-cli` and run `npm test`
- Ensure TypeScript compiles without errors: `npm run build`

### Development Flow
- **Build**: Navigate to component directory (e.g., `tasks/terraform-cli`) and run `npm run build` (runs TypeScript compiler)
- **Test**: Navigate to component directory and run `npm test` (runs Cucumber tests)
- **Test with Coverage**: Run `npm run test:coverage` for code coverage reports
- **Package**: From root, run `npm run pack-cli`, `npm run pack-inst`, or `npm run pack-views` for respective components

## Technology Stack

- **Language**: TypeScript (ES6, CommonJS modules)
- **Runtime**: Node.js (versions 20.x and 22.x supported)
- **Testing Framework**: Cucumber.js with cucumber-tsflow decorators
- **Assertion Library**: Chai
- **Build Tool**: TypeScript compiler (tsc)
- **Code Coverage**: nyc
- **Azure DevOps**: Uses Azure Pipelines Task SDK for task development
- **View Framework**: React (for terraform-plan view)

## Repository Structure

- `tasks/terraform-cli/`: Main CLI task implementation
  - `src/commands/`: Terraform command implementations (tf-init.ts, tf-plan.ts, tf-apply.ts, etc.)
  - `src/authentication/`: Authentication providers (Azure, AWS, GCS)
  - `src/backends/`: Backend configurations (AzureRM, AWS, GCS)
  - `src/providers/`: Provider configurations
  - `src/runners/`: Command execution builders
  - `src/tests/`: Cucumber feature files and step definitions
- `tasks/terraform-installer/`: Terraform installer task
- `views/terraform-plan/`: React-based UI view for displaying Terraform plans
- `.github/workflows/`: GitHub Actions CI workflows
- `pipelines/`: Azure DevOps pipeline definitions
- `scripts/`: Build and utility scripts

## Key Guidelines

### 1. Follow Existing Patterns
- Terraform commands follow a consistent pattern using the RunWithTerraform builder
- Look at existing command implementations (e.g., `src/commands/tf-validate.ts`) as templates
- Authentication and backend configuration use provider patterns
- Tests use Cucumber BDD style with Given/When/Then scenarios

### 2. Azure DevOps Task Development
- Tasks must follow Azure Pipelines Task SDK conventions
- Each task has a `task.json` manifest defining inputs, outputs, and metadata
- Use the `azure-pipelines-task-lib` for Azure DevOps-specific functionality
- Task versions follow semantic versioning in the task.json

### 3. Testing Requirements
- Write Cucumber feature files (`.feature`) for new functionality in BDD style
- Implement step definitions using cucumber-tsflow decorators (@given, @when, @then)
- Maintain test coverage for critical paths
- Tests should be isolated and not depend on external resources
- Use mocks for Azure DevOps task library interactions

### 4. TypeScript Conventions
- Use strict TypeScript compilation (`"strict": true`)
- Enable experimental decorators for cucumber-tsflow
- Target ES6 with CommonJS modules
- Prefer interfaces and type definitions for better type safety
- Use async/await for asynchronous operations

### 5. Terraform Command Support
- Support all major Terraform CLI commands (init, plan, apply, destroy, validate, fmt, etc.)
- Handle different backend types (AzureRM, AWS, GCS, local)
- Support multiple provider authentication methods (Azure, AWS, GCS)
- Handle secure file management for backend configurations

### 6. Documentation
- Update task READMEs when changing task behavior
- User-facing documentation is in `overview.md` at repository root
- Include YAML examples for Azure Pipeline usage
- Document environment variables and inputs in task manifests

### 7. CI/CD Integration
- **GitHub Actions**: Runs on PRs for unit tests and builds (`.github/workflows/ci.yml`)
- **Azure DevOps**: Handles integration tests and marketplace publishing (`pipelines/`)
- Tests run on Node.js 20.x and 22.x
- Integration tests require actual Azure DevOps environment

### 8. Version Management
- Version numbers are managed by GitVersion
- Use semantic versioning for releases
- Update version in task.json when making task changes

## Common Workflows

### Adding a New Terraform Command
1. Create command file in `tasks/terraform-cli/src/commands/` (e.g., `tf-newcommand.ts`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jason-johnson/azure-pipelines-tasks-terraform](https://github.com/jason-johnson/azure-pipelines-tasks-terraform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
