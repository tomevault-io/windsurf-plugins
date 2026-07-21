---
trigger: always_on
description: This document provides guidelines for AI agents (e.g., GitHub Copilot, MCP-based assistants, LLM-based tools) working with the Azure SDK for .NET repository. It defines safe and effective patterns for agent interactions with this codebase, automation workflows, and development processes.
---

# Azure SDK for .NET - AI Agent Guidelines

This document provides guidelines for AI agents (e.g., GitHub Copilot, MCP-based assistants, LLM-based tools) working with the Azure SDK for .NET repository. It defines safe and effective patterns for agent interactions with this codebase, automation workflows, and development processes.

## Repository Overview

### Purpose and Scope

The Azure SDK for .NET repository contains:
- **Data plane client Libraries**: SDKs for interacting with Azure services at application runtime
- **Management plane Libraries**: SDKs for provisioning and managing Azure resources
- **Code Generators**: Tools that generate Azure Data Plane and Management Plane SDKs
- **Build Infrastructure**: Common engineering systems and tooling for SDK development

### Repository Structure

```
/sdk                                                           # Individual Azure service SDKs
/eng/packages/http-client-csharp            # Azure Data Plane SDK generator
/eng/packages/http-client-csharp-mgmt # Azure Management Plane SDK generator
/eng                                                           # Build, test, and automation infrastructure
/doc                                                           # Documentation
```

For detailed developer instructions, see [CONTRIBUTING.md](https://github.com/Azure/azure-sdk-for-net/blob/main/CONTRIBUTING.md).

## Agent Interaction Guidelines

### Supported Agent Actions

AI agents may assist with the following activities:

#### Code Development
- **Reading and Understanding Code**: Navigating source files, understanding SDK patterns, and explaining implementations
- **Code Generation Support**: Assisting with SDK code generation using AutoRest and TypeSpec
- **Test Creation**: Writing unit tests and integration tests following existing patterns
- **Bug Fixes**: Identifying and fixing issues in SDK code
- **API Review**: Preparing code for API reviews and ensuring adherence to design guidelines

#### Documentation
- **README Updates**: Improving SDK documentation and code samples
- **Code Comments**: Adding inline documentation
- **Migration Guides**: Creating guides for breaking changes

#### Automation and Workflows
- **Build Verification**: Running builds and interpreting results
- **Test Execution**: Running test suites and analyzing failures
- **PR Triage**: Summarizing changes and checking CI status
- **Issue Analysis**: Interpreting bug reports and feature requests

### Safety Boundaries

AI agents **must not**:

- **Commit Secrets**: Never commit credentials, API keys, or sensitive configuration
- **Bypass Security**: Skip security checks or modify security-critical code without human review
- **Auto-merge PRs**: Merge pull requests without proper human approval
- **Modify CI/CD Pipelines**: Change GitHub Actions workflows without explicit permission
- **Delete Test Coverage**: Remove or disable existing tests unless explicitly instructed
- **Break API Compatibility**: Introduce breaking changes in GA libraries without explicit design approval

AI agents **should be cautious** when:

- Modifying generated code — **never** update generated code without running the generator to regenerate it
- Making changes to shared infrastructure in `/eng` — **never** do this unless explicitly asked
- Updating package dependencies (requires dependency management approval)
- Changing public APIs (requires API review)

## Key Workflows

### Building and Testing

#### Client Libraries

```powershell
# Build a specific service
cd sdk/eventhub
dotnet build

# Run tests (live tests are excluded by default)
dotnet test

# Run tests (explicitly skip live tests)
dotnet test --filter TestCategory!=Live

# Build and test via service.proj
dotnet build eng/service.proj /p:ServiceDirectory=eventhub
dotnet test eng/service.proj /p:ServiceDirectory=eventhub --filter TestCategory!=Live
```

#### Management Libraries

```powershell
# Build a specific management library
msbuild eng/mgmt.proj /p:scope=Compute

# Run tests
msbuild eng/mgmt.proj /t:RunTests /p:scope=Compute

# Create NuGet package
msbuild eng/mgmt.proj /t:CreateNugetPackage /p:scope=Compute
```

#### Full Repository Build

```powershell
# Build entire repository
dotnet build build.proj

# Build specific scope
dotnet build build.proj /p:Scope=servicebus
```

### Code Generation

#### Data Plane SDK Generation (AutoRest)

```powershell
# Generate code for a data plane SDK
cd sdk/<service>/<project>/src
dotnet build /t:GenerateCode -v d
```

#### Azure Generator (TypeSpec)

```powershell
# Install dependencies
cd eng/packages/http-client-csharp
npm install

# Generate test projects
./eng/scripts/Generate.ps1
```

#### Azure Management Generator

```powershell
# Install dependencies
cd eng/packages/http-client-csharp-mgmt
npm install

# Generate test projects
./eng/scripts/Generate.ps1
```

### API Review and Public API Changes

When making public API changes:

```powershell
# Export API for review
eng/scripts/Export-API.ps1 <service-directory>

# Example
eng/scripts/Export-API.ps1 tables

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure/azure-sdk-for-net](https://github.com/Azure/azure-sdk-for-net) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
