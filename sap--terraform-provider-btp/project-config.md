---
trigger: always_on
description: The repository implements the official (open source) Terraform Provider for SAP Business Technology Platform (SAP BTP). It enables infrastructure-as-code management of SAP BTP.
---

# Project Overview

The repository implements the official (open source) Terraform Provider for SAP Business Technology Platform (SAP BTP). It enables infrastructure-as-code management of SAP BTP.

Primary goals:
- Provide comprehensive, consistent CRUD / data source coverage of SAP BTP platform entities.
- Model SAP BTP primitives as Terraform resources and data sources with clear schemas, validation, and lifecycle handling.
- Offer predictable diffs and idempotent behavior; minimize drift and surface configuration errors early.
- Maintain high test coverage (unit + acceptance; VCR where feasible) and documentation parity (generated docs + examples).
- Adhere to HashiCorp Terraform Plugin Framework best practices (framework-based types, plan modifiers, validators, timeouts, proper diagnostics).

## Folder Structure

- `main.go`: Provider entrypoint (serves the Terraform provider server executable).
- `go.mod` / `go.sum`: Module definition; Go 1.25; declares HashiCorp plugin framework + supporting libs.
- `Makefile`: Standard developer commands (build, install, lint, generate, fmt, test, testacc).
- `README.md`: User-facing overview and badges.
- `DEVELOPER.md`: Local / container / Codespaces setup and dev overrides, committing guidance.
- `CONTRIBUTING.md`: Contribution process, DCO, AI usage guidelines.
- `sonar-project.properties`: SonarCloud static analysis configuration.
- `terraform-registry-manifest.json`: Registry publishing metadata.
- `examples/`: Practical Terraform configurations showcasing provider usage (provider, data-sources, resources subfolders).
- `docs/`: Generated provider documentation (data sources and resources). Do not hand-edit generated docs (use code + templates then `make generate`).
- `guides/`: Manually maintained topic guides (drift detection, import, sensitive data, etc.).
- `templates/`: Go text/template sources for generating documentation pages.
- `integration/`: Integration test/example Terraform configs (manual / scripted validation across resources).
- `regression-test/`: Additional Terraform scenarios for regression detection.
- `internal/`: Internal (non-exported) Go packages:
  - `btpcli/`: API client / facade for SAP BTP REST endpoints (abstraction layer over HTTP).
  - `tfutils/`: Shared Terraform + Go helper utilities (conversions, plan helpers, etc.).
  - `validation/`: Centralized schema validation helpers.
  - `version/`: Provider versioning utilities.
- `btp/provider/`: Core provider implementation code:
  - Resource files: `resource_<scope>_<entity>.go`
  - Data source files: `datasource_<scope>_<entity>.go`
  - Type models: `type_<scope>_<entity>.go` / variations for hierarchy models.
  - Tests: Paired `_test.go` files next to the implementation (unit and acceptance style). Data sources and resources each have a corresponding test file.
  - Helpers: `helper.go`, `helper_doc_values_format.go` for documentation value formatting and shared logic.
  - `provider.go` / `provider_test.go`: Provider schema and configuration implementation and tests.
  - `fixtures/`: (If present) serialized recordings / static payloads aiding tests (e.g., with go-vcr).

Naming scheme highlights inside `btp/provider/`:
- `datasource_<account|directory|subaccount|...>_<entity>.go`: Data source implementation.
- `resource_<account|directory|subaccount|...>_<entity>.go`: Resource implementation.
- `type_<scope><Entity>.go` or `type_<scope>_<entity>.go`: Strongly typed internal model or structure representing API data (used in schema translation).

## Libraries and Frameworks

Runtime and Framework:
- HashiCorp Terraform Plugin Framework (`github.com/hashicorp/terraform-plugin-framework`): Core abstraction for provider schema and CRUD.
- HashiCorp Terraform Plugin Go (`github.com/hashicorp/terraform-plugin-go`): Underlying protocol and plumbing.
- Framework Add-ons:
  - `terraform-plugin-framework-validators`: Attribute validators.
  - `terraform-plugin-framework-timeouts`: Declarative operation timeouts.

Testing and Quality:
- terraform-plugin-testing: Acceptance and unit test helpers.
- stretchr/testify: Assertions.
- go-vcr (`gopkg.in/dnaeon/go-vcr.v3`): Recording/replaying HTTP interactions (when applicable) to reduce live API dependency.
- golangci-lint: Aggregated linting.

Auxiliary:
- internal/btpcli: SAP BTP API client abstraction (respect API versioning and error mapping here first before resource layer changes).
- internal/validation: Central location for cross-schema validators.
- internal/version: Provider version injection (used in User-Agent strings, etc.).

## Coding Standards

### General Instructions

- Write simple, clear, and idiomatic Go code
- Favor clarity and simplicity over cleverness
- Follow the principle of least surprise
- Keep the happy path left-aligned (minimize indentation)
- Return early to reduce nesting
- Prefer early return over if-else chains; use if condition { return } pattern to avoid else blocks
- Make the zero value useful
- Write self-documenting code with clear, descriptive names
- Document exported types, functions, methods, and packages
- Use Go modules for dependency management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SAP/terraform-provider-btp](https://github.com/SAP/terraform-provider-btp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
