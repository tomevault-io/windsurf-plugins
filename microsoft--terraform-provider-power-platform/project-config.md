---
trigger: always_on
description: These instructions guide GitHub Copilot to follow our project's conventions and best practices when suggesting code. They cover how to format code, name resources and attributes, structure implementations, and write tests in this repository. By following these guidelines, Copilot's suggestions should align with the project's style and help contributors produce high-quality, consistent code. Always consider existing patterns in the repository—when in doubt, review similar resources or tests for r
---

# GitHub Copilot Custom Instructions – Terraform Provider Power Platform

These instructions guide GitHub Copilot to follow our project's conventions and best practices when suggesting code. They cover how to format code, name resources and attributes, structure implementations, and write tests in this repository. By following these guidelines, Copilot's suggestions should align with the project's style and help contributors produce high-quality, consistent code. Always consider existing patterns in the repository—when in doubt, review similar resources or tests for reference and keep the new code idiomatic to the project's practices.

## Development Setup & Workflow

- Use the provided **Makefile** commands for all build and test tasks:
  - `make install` to compile the provider code.
  - `make lint` to run linters and ensure code style compliance.
  - `make unittest` to run all unit tests (optionally use `TEST=<prefix>` to run tests matching a name prefix, e.g. `make unittest TEST=Environment` to run tests named with that prefix). This filters tests by regex `^(TestAcc|TestUnit)<prefix>`.
  - `make acctest TEST=<prefix>` to run acceptance tests (integration tests) matching a prefix. Always provide a specific test prefix to limit scope, and run these tests **only with user consent** (they run against real cloud resources). Note that `make acctest` automatically sets `TF_ACC=1` (no need to set it manually).
  - `make userdocs` to regenerate documentation
  - `make precommit` to run all checks once code is ready to commit. As a copilot agent you don't want to run this command as it will timeout for you. Read the makefile content and run needed commands manually.
  - `make coverage` to run all unit tests and output a code coverage report. It also shows the files that have changed on this branch to help target coverage suggestions to files in the current PR.
- Always run the above `make` commands from the repository root (e.g. in the `/workspaces/terraform-provider-power-platform` directory).
- **Never run** `terraform init` inside the provider repo. Terraform is only used in examples or tests; initializing in the provider directory is not needed and may cause conflicts.
- Do not manually edit files under the `/docs` folder. These files are auto-generated from the schema `MarkdownDescription` attributes. Instead, update schema's `MarkdownDescription` in code and run `make userdocs` to regenerate documentation.
- To try out an example configuration, navigate to its directory under `/examples` and run `terraform apply -auto-approve` (ensure you've built the provider and set it in your Terraform plugins path beforehand).

## File and Folder Structure

### Service Organization

- Organize all service implementations within the `internal/services` directory, with each service in its own subdirectory.
- Name service directories using lowercase words with underscores (e.g., `tenant_settings`, `environment_templates`).
- Choose service names that reflect the Power Platform domain they represent.

### Service Files

Each service directory MUST contain:

- **Models File**: Create a single `models.go` file containing all data models and DTO conversion functions.
- **API Client File**: Name as `api_<service_name>.go` (e.g., `api_licensing.go`).
- **Data Transfer Object File**: Create a single `dto.go` file containing all DTO objects used by the client to represent JSON sent to or received from the API
- **Test Files**: Name as `resource_<resource_name>_test.go` or `datasource_<data_source_name>_test.go` and place in the same directory. Must have a test file for every resource and data source file in a service.
- **Mock Data Files**: Place test JSON fixtures in `tests/resource/<test_scenario>/` or `tests/datasource/<test_scenario>/` subdirectories. Name JSON test files according to the pattern `<method>_<object>.json` (e.g., `get_environment.json`, `post_lifecycle.json`). JSON test files may have a numerical index if multiple API calls are made in a test scenario.

Each service SHALL contain one or more resources or data sources:

- **Resource Files**: Name as `resource_<resource_name>.go` (e.g., `resource_environment.go`).
- **Data Source Files**: Name as `datasource_<data_source_name>.go` (e.g., `datasource_tenant_settings.go`).

### Example Files

The `/examples` directory provides usage examples for the provider, resources, and data sources:

- Organize examples in three top-level directories:
  - `data-sources/` - Contains examples for all data source types
  - `resources/` - Contains examples for all resource types
  - `provider/` - Contains provider configuration examples

- Create a subdirectory for each resource or data source type under its respective category:
  - Name subdirectories exactly matching the resource/data source name (e.g., `powerplatform_environment`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/terraform-provider-power-platform](https://github.com/microsoft/terraform-provider-power-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
