---
trigger: always_on
description: This document provides guidance for AI agents, such as GitHub Copilot, when working with the Stryker.NET repository.
---

# AI Agent Instructions for Stryker.NET

This document provides guidance for AI agents, such as GitHub Copilot, when working with the Stryker.NET repository.

Reference these instruction files when applicable:
- **Always**: [Taming Copilot](.github/instructions/taming-copilot.instructions.md)
- **Always**: [Conventional Commit Guidelines](.github/instructions/conventional-commit.instructions.md)
- **Always**: [Security Best Practices](.github/instructions/security-and-owasp.instructions.md)
- **Always**: [Self-Explanatory Code Commenting](.github/instructions/self-explanatory-code-commenting.instructions.md)
- **When writing documentation**: [Markdown Instructions](.github/instructions/markdown.instructions.md)
- **When writing C#**: [C# Instructions](.github/instructions/csharp.instructions.md)
- **When working with Azure Pipelines**: [Azure DevOps Pipelines](.github/instructions/azure-devops-pipelines.instructions.md)
- **When working with GitHub Actions**: [GitHub Actions](.github/instructions/github-actions-ci-cd-best-practices.instructions.md)

Reference these skill files when applicable:
- **When running stryker on a project**: [Running Stryker](.github/skills/run-stryker/skill.md)

## Project Overview

Stryker.NET is a mutation testing framework for .NET projects. It allows you to test your tests by temporarily inserting bugs (mutations) in your source code to verify that tests catch them. Some of the key features of Stryker.NET include:

- Large set of built-in mutators for C# code
- Support for multiple test frameworks (xUnit, NUnit, MSTest, TUnit)
- Integration with Visual Studio Test platform
- Integration with the Microsoft Testing Platform (MTP)
- Detailed reporting of mutation testing results using [html and json report formats](https://github.com/stryker-mutator/mutation-testing-elements)
- Support for .NET Core and .NET Framework projects
- Reporting to the [Stryker Dashboard](https://dashboard.stryker-mutator.io/)
- Configurable mutation testing options and thresholds

Directory structure:
- `/src`: Main source code for Stryker.NET
  - `/src/Stryker.CLI`: Command-line interface for running Stryker.NET
  - `/src/Stryker.Core`: Core mutation testing engine. Contains the logic for analyzing projects, generating mutants, and reporting results.
  - `/src/Stryker.TestRunner`: Test runner integration for executing tests during mutation testing
  - `/src/Stryker.TestRunner.VsTest`: Test runner using the VsTest adapter for running tests with Visual Studio Test framework
  - `/src/Stryker.TestRunner.MicrosoftTestPlatform`: Test runner for Microsoft Testing Platform (MTP)
  - `/src/Stryker.Configuration`: Configuration and options management for Stryker.NET
  - `/src/Stryker.Abstractions`: Common interfaces and abstractions used across the project
  - `/src/Stryker.Utilities`: Utility functions and shared code used across the project
- `/docs`: Documentation for Stryker.NET
- `/integrationtest`: Integration tests for Stryker.NET
  - `/integrationtest/TargetProjects`: Target projects used for testing major features of Stryker.NET. For example different runtimes, test frameworks, and project types. These projects are used for testing Stryker.NET's core features.
  - `/integrationtest/Validation`: The tests validating the results of running Stryker.NET on the target projects
- `/ExampleProjects`: Example projects used only for testing F#

## Contributing Workflow

### Code Standards

- Follow the repository's `.editorconfig` and [Microsoft C# coding guidelines](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/inside-a-program/coding-conventions)
- Create or edit unit tests or integration tests for all changes
- Update documentation when adding features

### Pull Request Title Convention

When creating or updating pull requests, **always** use Angular-style conventional commit format for PR titles:
- Format: `<type>(<scope>): <subject>`
- Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`
- Scope: The file or group of files affected (optional but recommended)
- Subject: A short, imperative description (present tense)
- Example: `feat(mutators): add string mutator support`
- Example: `fix(cli): resolve configuration parsing issue`
- Example: `docs: update contributor guidelines`

**Why**: The project uses squash merging, so the PR title becomes the commit message in the main branch history.

### Running Tests

- **Unit tests**: Use #tool:execute/runTests or when the tool is not available run `dotnet test`
- **Integration tests**:
  - On **Windows**: Run `.\integration-tests.ps1` in the root of the repo (PowerShell 7 recommended)
  - On **macOS/Linux**: Run `pwsh ./integration-tests.ps1` in the root of the repo (requires [PowerShell 7](https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell))
- Always run unit tests and integration tests after making a change

### Testing Locally


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stryker-mutator/stryker-net](https://github.com/stryker-mutator/stryker-net) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
