---
trigger: always_on
description: This document provides guidance for AI agents (e.g., GitHub Copilot, MCP servers, or LLM-based assistants) interacting with the Azure SDK for Rust repository.
---

# AGENTS.md

This document provides guidance for AI agents (e.g., GitHub Copilot, MCP servers, or LLM-based assistants) interacting with the Azure SDK for Rust repository.

## Repository Overview

The Azure SDK for Rust provides Rust language bindings and client libraries for Azure services, following the [Azure SDK Design Guidelines for Rust](https://azure.github.io/azure-sdk/rust_introduction.html).

- **Primary Language**: Rust
- **Minimum Supported Rust Version (MSRV)**: found in the root `Cargo.toml` file
- **Key Technologies**: Cargo, TypeSpec, OpenTelemetry, Test Proxy

## Repository Structure

```text
.
├── sdk/                      # Service-specific crates organized by service
│   └── {service}/            # Service directory (e.g., "keyvault", "storage")
│       ├── {crate}/          # Service crate (e.g., "azure_security_keyvault_secrets")
│       ├── assets.json       # Pointer to test recordings (may be under {crate}/)
│       ├── test-resources.bicep # Test resource definitions (may be under {crate}/)
│       └── tsp-location.yaml # Pointer to TypeSpec in azure-rest-api-specs (may be under {crate}/)
├── eng/                      # Engineering system scripts and common tooling
├── doc/                      # Additional documentation
├── .github/
│   ├── instructions/         # Agent instruction files for specific tasks
│   └── skills/               # Copilot skills (e.g., check-spelling, lint-markdown)
├── CONTRIBUTING.md           # Contribution guidelines (see for detailed workflows)
└── README.md                 # Repository overview
```

## Agent Capabilities

Always check if there is an MCP tool or skill available before performing operations manually, including listing Azure subscriptions, deploying resources, setting up a new crate, generating code, and other common workflows.

All new crates must be generated from TypeSpec specifications in [Azure/azure-rest-api-specs]. TypeSpec specifications are located under `specification/{service}/`. Use the `create-crate` skill to set up a new crate.

### Recommended Actions

AI agents can assist with:

1. **Code Generation**
   - Writing new Rust code following the coding conventions below
   - Generating unit tests using `#[cfg(test)]` modules
   - Creating integration tests with `#[recorded::test]` attributes (see [CONTRIBUTING.md](CONTRIBUTING.md) for details)
   - Generating documentation tests in `.rs` files (avoid `no_run` when tests can be run)
   - Running `cargo fmt` and `cargo clippy` on all modified crates (see [Linting and Formatting](#linting-and-formatting))

2. **Code Review**
   - Identifying potential bugs or safety issues
   - Suggesting improvements for idiomatic Rust patterns
   - Checking adherence to Azure SDK design guidelines
   - Reviewing error handling using `azure_core::Result<T>`

3. **Documentation**
   - Improving inline documentation (using `///` doc comments)
   - Updating README files (use ` ```rust no_run` for examples with placeholders)
   - Creating or updating CHANGELOG entries (see [.github/instructions/changelog.instructions.md](.github/instructions/changelog.instructions.md))
   - Writing hero scenario examples in doc comments (avoid examples in `examples/` directories unless demonstrating primary use cases)

4. **Issue Triage**
   - Labeling issues with appropriate tags
   - Identifying duplicate issues
   - Suggesting relevant code owners based on `CODEOWNERS`
   - Summarizing issue discussions

5. **Refactoring**
   - Applying clippy suggestions
   - Improving code organization and modularity
   - Updating dependencies in `Cargo.toml`
   - Consolidating imports (e.g., `use std::{borrow::Cow, marker::PhantomData};` instead of separate lines)

### Restricted Actions

AI agents **should not**:

1. **Modify Generated Code**
   - Never edit files in `generated/` subdirectories
   - These are produced by TypeSpec code generators and will be overwritten
   - Instead, propose changes to TypeSpec specifications in [Azure/azure-rest-api-specs]

2. **Break API Compatibility**
   - Avoid introducing breaking changes without explicit approval
   - Check if changes affect public APIs before proceeding
   - Consider the deprecation process (see [doc/deprecation-process.md](doc/deprecation-process.md))

3. **Bypass CI/CD Checks**
   - Do not suggest skipping or disabling CI checks
   - All code must pass `cargo build`, `cargo test`, and `cargo clippy`

4. **Commit Secrets**
   - Never include credentials, keys, or tokens in code
   - Use environment variables for sensitive data
   - Sanitize test recordings to remove secrets

5. **Modify Security or License Files**
   - Do not alter `SECURITY.md`, `LICENSE.txt`, or `CODE_OF_CONDUCT.md` without maintainer approval

6. **Hand-Write Generated Clients**
   - Do not hand-write client, model, or operation code when a TypeSpec specification exists in [Azure/azure-rest-api-specs]
   - Use `tsp-client update` or the `azsdk_package_generate_code` MCP tool to generate client code from TypeSpec

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure/azure-sdk-for-rust](https://github.com/Azure/azure-sdk-for-rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
