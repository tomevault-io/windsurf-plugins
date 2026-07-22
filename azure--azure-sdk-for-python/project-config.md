---
trigger: always_on
description: This file describes how AI agents (e.g., GitHub Copilot, MCP servers, or LLM-based assistants) should interact with this repository.
---

# AGENTS.md - Azure SDK for Python

This file describes how AI agents (e.g., GitHub Copilot, MCP servers, or LLM-based assistants) should interact with this repository.

## Repository Overview

**Purpose**: This repository contains the active development of the Azure SDK for Python, providing client libraries and management libraries for Azure services.

**Scope**: 
- 100+ Python packages for Azure services
- Client libraries for data plane operations
- Management libraries for Azure Resource Manager (ARM)
- Shared core functionality (authentication, retries, logging, transport)
- TypeSpec-based SDK generation from API specifications
- Comprehensive testing, validation, and documentation infrastructure

**Main Branch**: `main`

**Key Documentation**:
- [Azure SDK Python Design Guidelines](https://azure.github.io/azure-sdk/python_design.html)
- [Contributing Guide](https://github.com/Azure/azure-sdk-for-python/blob/main/CONTRIBUTING.md)
- [Developer Documentation](https://github.com/Azure/azure-sdk-for-python/tree/main/doc/dev)

## Repository Structure

```
azure-sdk-for-python/
├── sdk/                    # Service-specific libraries (e.g., sdk/storage/, sdk/ai/)
├── eng/                    # Engineering system tools and configurations
├── doc/                    # Developer documentation and guides
├── scripts/                # Automation scripts
├── .github/                # GitHub workflows, Copilot instructions, skills, and prompts
│   ├── copilot-instructions.md  # Detailed Copilot-specific guidance
│   ├── skills/             # Reusable Copilot skills (e.g., find-package, fix-pylint)
│   └── prompts/            # Copilot prompt files for common tasks
└── tools/                  # Development tools
```

## AI Agent Guidelines

### Supported Actions

AI agents can assist with the following activities:

#### Code Generation & Development
- **TypeSpec SDK Generation**: Generate Python SDKs from TypeSpec specifications
  - Follow the complete workflow in `.github/copilot-instructions.md`
  - Use MCP tools for environment verification, code generation, and validation
  - Time estimate: 10-15 minutes for full generation workflow
  
- **Code Fixes**: Address linting, type checking, and validation issues
  - Pylint, MyPy, Pyright, Verifytypes, Sphinx warnings
  - Follow official guidelines and existing patterns
  - Make minimal, surgical changes

- **Documentation**: Update CHANGELOG.md, README files, and API documentation
  - Follow existing formatting conventions
  - Include version information and release dates

#### Testing & Validation
- **Run Tests**: Execute test suites using the `azpysdk` test runner
  - Unit tests, integration tests, samples
  - Support both local and CI environments
  
- **Static Analysis**: Run and fix issues from:
  - Pylint (code quality)
  - MyPy, Pyright (type checking)
  - Bandit (security)
  - Black (formatting)
  - Sphinx (documentation)

#### Pull Request Management
- **PR Creation**: Create draft PRs with descriptive titles and descriptions
- **PR Review**: Analyze PR feedback and make requested changes
- **Status Checks**: Monitor CI/CD pipeline status and address failures

#### Issue Triage & Labeling
- **Issue Analysis**: Review and categorize issues
- **Service Labels**: Validate and create service labels (e.g., `Azure.AI.Projects`)
- **CODEOWNERS**: Validate and update CODEOWNERS entries

### Relationship to Copilot Instructions

This `AGENTS.md` file is the primary reference for AI agents working in this repository. It follows the [AGENTS.md standard](https://github.com/agentsmd/agents.md) that most AI coding tools recognize automatically. For additional Copilot-specific guidance (e.g., TypeSpec generation details, MCP tool usage), see [`.github/copilot-instructions.md`](https://github.com/Azure/azure-sdk-for-python/blob/main/.github/copilot-instructions.md). Reusable skills are available in [`.github/skills/`](https://github.com/Azure/azure-sdk-for-python/tree/main/.github/skills/) and prompt templates in [`.github/prompts/`](https://github.com/Azure/azure-sdk-for-python/tree/main/.github/prompts/).

### Key Workflows

#### 1. TypeSpec SDK Generation Workflow

**Prerequisites**:
- GitHub CLI authenticated (`gh auth login`)
- Feature branch (not `main`)
- PowerShell installed (for MCP tools)
- Python virtual environment active

**Steps**:
1. **Environment Verification** - Use `verify_setup` MCP tool
2. **SDK Generation** - Use azure-sdk-python-mcp generation tools (~2 minutes)
3. **Static Validation** - Run sequential validation steps (~3-5 minutes each):
   - Pylint, MyPy, Pyright, Verifytypes
   - Sphinx, Mindependency, Bandit, Black
   - Samples, Breaking changes
4. **Documentation Update** - Update CHANGELOG.md and version files
5. **Commit & Push** - Stage, commit, and push changes
6. **PR Creation** - Create draft PR with generated description
7. **Handoff** - Provide PR link for azure-rest-api-specs agent

**Estimated Time**: 10-15 minutes

#### 2. Code Quality Workflow

**Running Validation**:
```bash
# Use azpysdk with appropriate environment from the package directory
azpysdk <environment> .

# Examples:
azpysdk pylint .
azpysdk mypy .
```

**Fixing Issues**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure/azure-sdk-for-python](https://github.com/Azure/azure-sdk-for-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
