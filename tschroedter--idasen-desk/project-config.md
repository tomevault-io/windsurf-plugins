---
trigger: always_on
description: This repository contains a Windows 10/11 desktop application for controlling Ikea's Idasen standing desk via Bluetooth LE. The application provides a system tray interface with features like preset heights, hotkey support, and automatic connection management.
---

# Copilot Instructions for idasen-desk

## Project Overview

This repository contains a Windows 10/11 desktop application for controlling Ikea's Idasen standing desk via Bluetooth LE. The application provides a system tray interface with features like preset heights, hotkey support, and automatic connection management.

**Technology Stack:**
- C# / .NET 8.0
- WPF (Windows Presentation Foundation)
- Target Framework: `net8.0-windows10.0.19041`
- Dependency Injection: Autofac
- Bluetooth LE for desk communication
- Testing: xUnit, NSubstitute, FluentAssertions

## Repository Structure

```
├── .github/                    # GitHub workflows and configurations
│   ├── workflows/             # CI/CD pipelines
│   │   ├── dotnet-ci.yml     # Build, test, and release workflow
│   │   ├── release-drafter.yml # Automated release notes
│   │   └── update-changelog.yml # Changelog updates
│   ├── release-drafter.yml    # Release drafter configuration
│   └── pull_request_template.md
├── docs/                       # Documentation
│   ├── CHANGELOG_AUTOMATION.md
│   ├── IMPLEMENTATION_SUMMARY.md
│   └── README.md
├── scripts/                    # Helper scripts
│   └── create-labels.sh       # PR label creation script
├── src/                        # Source code
│   ├── Idasen.SystemTray.Win11/        # Main application
│   │   ├── ViewModels/        # MVVM ViewModels
│   │   ├── Views/             # WPF Views
│   │   ├── Services/          # Application services
│   │   ├── Helpers/           # Helper classes
│   │   └── Utils/             # Utility classes
│   └── Idasen.SystemTray.Win11.Tests/  # Unit tests
└── CHANGELOG.md               # Auto-generated changelog
```

## Building and Testing

### Prerequisites
- .NET 8.0 SDK
- Windows 10/11 (for building and running)
- Visual Studio 2022 or VS Code with C# extension

### Build Commands

```bash
# Navigate to source directory
cd src

# Restore dependencies
dotnet restore Idasen-Desk.sln

# Build solution (Release)
dotnet build Idasen-Desk.sln --configuration Release

# Build solution (Debug)
dotnet build Idasen-Desk.sln --configuration Debug

# Run tests
dotnet test Idasen-Desk.sln --configuration Release --verbosity normal

# Publish self-contained executable
dotnet publish Idasen.SystemTray.Win11/Idasen.SystemTray.Win11.csproj \
  --configuration Release \
  --runtime win-x64 \
  --self-contained true \
  -p:PublishSingleFile=true \
  -p:PublishReadyToRun=true
```

### Running Tests

All tests use xUnit framework and should pass before submitting PRs:

```bash
cd src
dotnet test --no-build --verbosity normal
```

## Code Style and Standards

### General Guidelines

1. **Warnings as Errors**: The project treats warnings as errors (`<TreatWarningsAsErrors>true</TreatWarningsAsErrors>`)
2. **Nullable Reference Types**: Enabled throughout the project
3. **Code Analysis**: 
   - .NET analyzers are enabled with `latest-recommended` analysis level
   - Code style is enforced during build
   - Analyzers run during both build and live analysis

### Naming Conventions

- Follow standard C# naming conventions
- Use PascalCase for classes, methods, and properties
- Use camelCase for local variables and parameters
- Prefix interfaces with `I` (e.g., `ISettingsService`)
- Use meaningful, descriptive names

### Testing Guidelines

- Test files mirror the structure of the main project
- Use xUnit for test framework
- Use NSubstitute for mocking
- Use FluentAssertions for assertions
- Test class names should end with `Tests`
- Test method names should clearly describe what is being tested (underscore separation is allowed for tests - see `NoWarn CA1707`)

### Code Organization

- Follow MVVM pattern for UI components
- Use dependency injection via Autofac
- Keep ViewModels testable and UI-framework independent
- Services should be registered in the IoC container
- Use interfaces for testability and loose coupling

## Pull Request Guidelines

### Required PR Labels

All PRs **must** have at least one changelog label. This is crucial for our automated changelog system:

**Change Type Labels** (choose at least one):
- `feature` or `enhancement` - New features and enhancements
- `fix`, `bugfix`, or `bug` - Bug fixes
- `chore` or `maintenance` - Maintenance tasks, refactoring, dependency updates
- `documentation` or `docs` - Documentation changes
- `security` - Security-related changes

**Version Bump Labels** (optional, defaults to `patch`):
- `major` - Breaking changes (x.0.0)
- `minor` - New features (0.x.0)
- `patch` - Bug fixes (0.0.x) - **default**

### PR Checklist

Before submitting a PR, ensure:
- [ ] Code follows the project's code style
- [ ] Self-review completed
- [ ] Code is commented where necessary (especially complex logic)
- [ ] Documentation updated if applicable
- [ ] No new warnings introduced
- [ ] Tests added/updated for new functionality
- [ ] All tests pass locally
- [ ] At least one changelog label added

### Setting Up PR Labels

If you're a maintainer and need to create the required labels:

```bash
# Install GitHub CLI if needed
gh auth login


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tschroedter/idasen-desk](https://github.com/tschroedter/idasen-desk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
