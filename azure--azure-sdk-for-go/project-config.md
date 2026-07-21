---
trigger: always_on
description: This file provides guidance for AI agents (e.g., GitHub Copilot, MCP, or LLM-based assistants) interacting with the Azure SDK for Go repository.
---

# AGENTS.md

This file provides guidance for AI agents (e.g., GitHub Copilot, MCP, or LLM-based assistants) interacting with the Azure SDK for Go repository.

## Repository Overview

For a comprehensive overview of this repository, see the [README.md](https://github.com/Azure/azure-sdk-for-go/blob/main/README.md).

### Key Documentation
- [Main README](https://github.com/Azure/azure-sdk-for-go/blob/main/README.md) - Getting started, package information, and repository structure
- [Contributing Guide](https://github.com/Azure/azure-sdk-for-go/blob/main/CONTRIBUTING.md) - Contribution guidelines and PR requirements
- [Developer Setup](https://github.com/Azure/azure-sdk-for-go/blob/main/documentation/development/setup.md) - Environment setup for SDK development
- [Release Documentation](https://github.com/Azure/azure-sdk-for-go/blob/main/documentation/development/release.md) - Package release process
- [Copilot Instructions](https://github.com/Azure/azure-sdk-for-go/blob/main/.github/copilot-instructions.md) - Copilot-specific guidance

### Go Version Support
The SDK is compatible with the two most recent major Go releases, following Go's official [support policy](https://go.dev/doc/devel/release#policy).

## Agent Capabilities and Boundaries

### Supported Actions

AI agents can assist with the following activities:

#### Code Development
- **Reading and understanding code**: Browse SDK packages, understand APIs, and explain functionality
- **Code suggestions**: Propose improvements, bug fixes, or new features following [Azure Go SDK Guidelines](https://azure.github.io/azure-sdk/golang_introduction.html)
- **Testing**: Write or update unit tests using `github.com/stretchr/testify/require`
- **Examples**: Create example code in `example*_test.go` files following the [Go examples guidelines](https://github.com/Azure/azure-sdk-for-go/blob/main/.github/instructions/go-examples.instructions.md)

#### Documentation
- **README updates**: Improve module READMEs and documentation
- **Code comments**: Add or improve GoDoc comments following [documentation style](https://azure.github.io/azure-sdk/golang_introduction.html#documentation-style)
- **CHANGELOG updates**: Document changes in CHANGELOG.md files

#### Issue and PR Management
- **Issue triage**: Review issues, suggest labels, identify duplicates
- **PR review assistance**: Analyze PRs, suggest improvements, check for guideline compliance
- **Question answering**: Help developers with SDK usage questions

### Automation Boundaries

AI agents should **NOT** perform the following actions without human approval:

#### Build and Release
- **Triggering releases**: Only humans should use `CheckPackageReleaseReadiness` and `ReleasePackage` MCP tools
- **Modifying CI/CD pipelines**: Changes to `ci.yml`, Azure Pipelines configurations, or workflow files require careful review
- **Approving releases**: Release stage approvals in pipelines must be done by authorized humans

#### Code Generation
- **Regenerating SDK code**: Most packages in `sdk/` are generated from [Azure API specs](https://github.com/Azure/azure-rest-api-specs) using TypeSpec and should not be manually modified
- **TypeSpec changes**: SDK generation from specifications requires specific tools and workflows (see [code generation docs](https://github.com/Azure/azure-sdk-for-go/blob/main/documentation/development/generate.md))

#### Security and Compliance
- **Security issues**: Must be reported privately to <secure@microsoft.com>, not in public issues
- **License changes**: No modifications to licensing without explicit approval

## Key Workflows

### Development Workflow

```bash
# Navigate to the SDK module you want to work with
cd sdk/azcore  # or any other module

# Build the module
go build ./...

# Run tests
go test ./...
```

### TypeSpec/Code Generation Workflow

For modules with `tsp-location.yaml`:

```bash
# Install prerequisites
npm install -g @typespec/compiler
npm install -g @azure-tools/typespec-client-generator-cli

# Navigate to the module directory
cd sdk/<service>/<module>

# Regenerate the SDK from TypeSpec
tsp-client update
```

See [TypeSpec location instructions](https://github.com/Azure/azure-sdk-for-go/blob/main/.github/instructions/tsp-location.instructions.md) for details.

### Contributing Workflow

For detailed contribution guidelines, see [CONTRIBUTING.md](https://github.com/Azure/azure-sdk-for-go/blob/main/CONTRIBUTING.md).

Key steps:
1. **Fork and clone** the repository
2. **Create a feature branch** from `main`
3. **Make changes** following the [Go SDK Guidelines](https://azure.github.io/azure-sdk/golang_introduction.html)
4. **Add tests** to ensure CI catches future regressions
5. **Update documentation** (README, CHANGELOG, examples)
6. **Run tests locally** to verify changes
7. **Submit a PR** with a descriptive title and reference to related issues
8. **Address review feedback** in additional commits

### PR Review Checklist

For comprehensive PR requirements, see the [Pull Requests section in CONTRIBUTING.md](https://github.com/Azure/azure-sdk-for-go/blob/main/CONTRIBUTING.md#pull-requests).

Agents can help verify PRs meet key requirements:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure/azure-sdk-for-go](https://github.com/Azure/azure-sdk-for-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
