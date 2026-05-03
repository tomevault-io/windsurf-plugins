---
trigger: always_on
description: Copilot instructions for CodeMedic - A .NET repository health analysis tool
---


# CodeMedic Copilot Instructions

## Project Overview

**CodeMedic** is a .NET 10 console application that provides comprehensive health analysis and reporting for .NET repositories. The application scans and reviews .NET projects to deliver actionable insights about dependency health, architecture quality, code standards, and repository maintenance.

### Technology Stack
- **Framework**: .NET 10 (.NET 10.0)
- **Application Type**: Console/CLI Application
- **Language**: C#
- **Primary Delivery**: Cross-platform command-line interface

## Core Principles

### 1. Cross-Platform Console UI
- Ensure all UI components work consistently across **Windows, macOS, and Linux**
- Use cross-platform CLI libraries that abstract OS-specific concerns
- Test and validate output formatting on all target operating systems
- Prefer libraries like **Spectre.Console** for rich terminal output that works everywhere

### 2. Console-First Design
- Design all features with the command-line interface as the primary interaction model
- Provide clear, actionable command-line feedback and output
- Support piping and scripting patterns where applicable
- Implement consistent command structure and help documentation

### 3. Health Analysis Focus
CodeMedic delivers features that help developers:
- **Scan** .NET repositories for health indicators
- **Review** dependencies, architecture, code quality, and build health
- **Report** findings in human-readable and machine-readable formats

## Key Features

### Repository Health Dashboard
A unified, extensible command that orchestrates multiple subsystems and produces a holistic, actionable report. The dashboard aggregates:
- Overall health score and trends
- Code quality summary
- Architecture and layering analysis
- Dependency health status
- Build performance metrics
- Test coverage and health
- Security posture and configuration
- GitHub workflow activity
- Doctor's Orders (actionable recommendations)

### Bill of Materials (BOM)
A comprehensive, multi-layered inventory of:
- NuGet packages (direct and transitive dependencies)
- Framework and platform features
- External services and vendors
- Environmental requirements
- Risk factors and known vulnerabilities
- License compliance information

Outputs available in both human-readable and machine-readable formats (JSON, Markdown).

## Development Guidelines

### Architecture
- Follow **modular design patterns** to support extensibility
- Separate concerns: scanning engines, analysis processors, report generators
- Make the BOM and other analysis engines pluggable subsystems
- Design for future feature additions without breaking existing functionality

### Command-Line Libraries
- Use **Spectre.Console** for rich formatting, tables, and interactive prompts
- Implement **System.CommandLine** or similar for robust command parsing
- Consider **Figlet** for ASCII art headers and branding
- Use progress indicators and spinners for long-running operations
- Provide color-coded output that degrades gracefully on systems without color support

### Code Quality Standards
- Enable **Nullable reference types** (`<Nullable>enable</Nullable>`)
- Use **implicit usings** (`<ImplicitUsings>enable</ImplicitUsings>`)
- Follow C# naming conventions and best practices
- Activate TreatWarningsAsErrors so that we can follow good coding practices
- Write defensive code with proper error handling
- Write unit-testable code as much as possible
- Provide meaningful error messages to end users

### Versioning
- Use **Nerd Bank Git Versioning** (NBGv2) to manage project version numbers
- Version is automatically calculated from git commits and tags
- Configure versioning in `.version.json` at the repository root
- Version information is injected into assemblies automatically
- Semantic versioning is enforced through git workflow

### Output and Reporting
- Default to human-readable console output
- Support machine-readable formats (JSON) for automation and integration
- Include structured logging for debugging and troubleshooting
- Ensure output is accessible and clear regardless of terminal capabilities

## Code Organization

### Project Structure
```
CodeMedic/
├── Program.cs                 # Application entry point
├── Commands/                  # Command definitions
├── Engines/                   # Analysis engines (BOM, Health, etc.)
├── Models/                    # Data models and entities
├── Processors/                # Analysis and transformation logic
├── Reporters/                 # Output formatters and generators
├── Utilities/                 # Cross-cutting utilities
└── Plugins/                   # Plugin infrastructure
    └── PluginLoader.cs        # Plugin discovery and lifecycle management
```

### Naming Conventions
- Command classes: `{Feature}Command.cs` (e.g., `HealthCommand.cs`, `BomCommand.cs`)
- Engine classes: `{Feature}Engine.cs` (e.g., `BomEngine.cs`)
- Model classes: PascalCase (e.g., `Package.cs`, `HealthReport.cs`)
- Plugin interfaces: `I{Feature}Plugin.cs` (e.g., `IAnalysisEnginePlugin.cs`)

## Plugin Architecture

CodeMedic supports a plugin system that allows extending analysis capabilities, adding custom commands, and developing specialized reporters without modifying core code.

### Core Plugin Interfaces

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FritzAndFriends/codemedic](https://github.com/FritzAndFriends/codemedic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
