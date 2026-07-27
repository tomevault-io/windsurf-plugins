---
trigger: always_on
description: Microsoft Application Inspector is a source code characterization tool that identifies coding features based on well-known library/API calls using regex patterns and rules. It supports multiple programming languages and generates reports in HTML, JSON, SARIF, and text formats.
---

# GitHub Copilot Instructions for ApplicationInspector

## Project Overview

Microsoft Application Inspector is a source code characterization tool that identifies coding features based on well-known library/API calls using regex patterns and rules. It supports multiple programming languages and generates reports in HTML, JSON, SARIF, and text formats.

## Tech Stack

- **Language**: C# (supports netstandard2.1; .NET 8.0+ for executables; aims to support all currently supported .NET versions)
- **Build System**: dotnet CLI
- **Test Framework**: xUnit
- **Solution Structure**: Multi-project .NET solution with 7 projects
- **Rule Format**: JSON files following rule-schema-v1.json schema definition in repository root

## Project Structure

- `AppInspector.CLI/` - Command-line interface application
- `AppInspector/` - Core commands library (AppInspector.Commands.csproj)
- `AppInspector.RulesEngine/` - Rule processing and pattern matching engine
- `AppInspector.Common/` - Common utilities and types
- `AppInspector.Logging/` - Logging infrastructure
- `AppInspector.Tests/` - Test suite using xUnit
- `AppInspector.Benchmarks/` - Performance benchmarks
- `AppInspector/rules/default/` - Built-in detection rules organized by category

## Build and Test Commands

### Important Build Notes

**Version Tracking**: This project uses Nerdbank.GitVersioning (nbgv) for version tracking, which requires a **deep clone** of the repository. Shallow clones will not work correctly with version generation.

**NuGet Packages**: The repository uses a custom `nuget.config` that points to a private Azure DevOps feed. If you need to restore packages and don't have access to the private feed, you may need to temporarily modify `nuget.config` to pull from nuget.org instead:
```xml
<packageSources>
  <clear />
  <add key="nuget.org" value="https://api.nuget.org/v3/index.json" />
</packageSources>
```
**IMPORTANT**: Never check in changes to `nuget.config`. The private feed configuration should remain in the repository.

### Building
```bash
# Build debug version
dotnet build

# Build release version
dotnet build -c Release

# Platform-specific publish
dotnet publish -c Release -r win10-x64
dotnet publish -c Release -r linux-x64
dotnet publish -c Release -r osx-x64
```

### Testing
```bash
# Run all tests
dotnet test

# Run specific test project
dotnet test AppInspector.Tests/AppInspector.Tests.csproj

# Run tests with verbose output
dotnet test --verbosity normal
```

### Running the CLI
```bash
# Analyze code
dotnet run --project AppInspector.CLI -- analyze -s path/to/source

# Verify rules
dotnet run --project AppInspector.CLI -- verifyrules -r path/to/rules

# Export tags
dotnet run --project AppInspector.CLI -- exporttags
```

## Coding Standards and Conventions

### General C# Guidelines

- Follow standard C# naming conventions (PascalCase for public members, camelCase for private fields)
- Use meaningful, descriptive names for variables, methods, and classes
- Keep methods focused and single-purpose
- Add XML documentation comments for public APIs
- Use nullable reference types where appropriate
- Prefer `var` for local variables when the type is obvious
- Use LINQ for collection operations when it improves readability

### Specific Project Conventions

- **Namespace Structure**: Follow the project name structure (e.g., `Microsoft.ApplicationInspector.Commands`)
- **Logging**: Use the logging infrastructure from `AppInspector.Logging` namespace
- **Error Handling**: Return structured result types rather than throwing exceptions for expected failures
- **Async/Await**: Use async methods for I/O operations; follow the Task-based Asynchronous Pattern (TAP)

## Rule Development Guidelines

### Rule File Structure

Rules are JSON files located in `AppInspector/rules/default/` organized by category:
- `cryptography/` - Cryptographic operations
- `security_feature/` - Security-related features (authentication, authorization, etc.)
- `data_handling/` - Database, serialization, deserialization
- `os/` - Operating system operations (file I/O, process execution, reflection)
- `cloud_services/` - Cloud platform APIs
- `frameworks/` - Framework detection
- And more...

### Rule Schema Requirements

The rule schema is defined in `rule-schema-v1.json` in the repository root.

Each rule must include:
- `id` - Unique identifier (format: AI######, e.g., AI038900)
- `name` - Human-readable name
- `tags` - Array of tags describing the feature (e.g., "Authentication.Microsoft.Online")
- `patterns` - Array of pattern objects for detection
- `description` - Optional detailed description
- `severity` - Optional: Critical, Important, Moderate, BestPractice, ManualReview, Unspecified
- `applies_to` - Optional: array of language identifiers (e.g., ["csharp", "java"])

### Pattern Types

- `string` - Literal string match
- `regex` - Regular expression
- `regexword` - Regex with word boundaries automatically added
- `substring` - Substring match

### Pattern Best Practices


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/ApplicationInspector](https://github.com/microsoft/ApplicationInspector) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
