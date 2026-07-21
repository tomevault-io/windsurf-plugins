---
trigger: always_on
description: This document provides instructions for LLM agents working with the Orchard Core codebase. It covers building, testing, and creating new features.
---

# Orchard Core - Agent Guidelines

This document provides instructions for LLM agents working with the Orchard Core codebase. It covers building, testing, and creating new features.

## Project Overview

Orchard Core is an open-source, modular, multi-tenant application framework and CMS for ASP.NET Core. It consists of:

- **Orchard Core Framework**: An application framework for building modular, multi-tenant applications
- **Orchard Core CMS**: A Web Content Management System built on top of the framework

**Repository**: <https://github.com/OrchardCMS/OrchardCore>  
**Documentation**: <https://docs.orchardcore.net/>

## Prerequisites

- **.NET SDK**: Version 10.0+ (see `global.json` for exact version requirements)
- **Node.js**: Version 22.x (for asset compilation)
- **Yarn**: Version 4.x (package manager for frontend assets)

## Building the Project

### Command Line

```bash
# Navigate to the web project
cd src/OrchardCore.Cms.Web

# Run the application
dotnet run

# Or build with specific configuration
dotnet build -c Debug -f net10.0
```

### Full Solution Build

```bash
# From repository root
dotnet build OrchardCore.sln

# Build with Release configuration
dotnet build OrchardCore.sln -c Release
```

### Target Framework

The default target framework is `net10.0` as defined in `src/OrchardCore.Build/TargetFrameworks.props`.

## Running the Application

```bash
cd src/OrchardCore.Cms.Web
dotnet run -f net10.0
```

The application will be available at `http://localhost:5000` (and `https://localhost:5001`).

## Running Tests

### Unit Tests

Unit tests are located in the `test/` directory and use xUnit v3.

```bash
# Run all tests
dotnet test

# Run tests for a specific project
dotnet test test/OrchardCore.Tests/OrchardCore.Tests.csproj

# Run tests with a filter
dotnet test --filter-method "*.YourTest"
```

Arguments for tests:

```bash
--filter-class
```

Run all methods in a given test class. Pass one or more fully qualified type names (i.e.,
'MyNamespace.MyClass' or 'MyNamespace.MyClass+InnerClass'). Wildcard '*' is supported at
the beginning and/or end of each filter.
    Note: Specifying more than one is an OR operation.
        This is categorized as a simple filter. You cannot use both simple filters and query filters.

```bash        
--filter-method
```

Run a given test method. Pass one or more fully qualified method names (i.e.,
'MyNamespace.MyClass.MyTestMethod'). Wildcard '*' is supported at the beginning and/or end
of each filter.
    Note: Specifying more than one is an OR operation.
        This is categorized as a simple filter. You cannot use both simple filters and query filters.


### Functional Tests (Playwright)

End-to-end tests are located in `test/OrchardCore.Tests.Functional/`.

```bash
# Run CMS functional tests
dotnet test test/OrchardCore.Tests.Functional/OrchardCore.Tests.Functional.csproj --filter-class "*Cms*"

# Run MVC functional tests
dotnet test test/OrchardCore.Tests.Functional/OrchardCore.Tests.Functional.csproj --filter-class "*Mvc*"
```

### Automated Browser Testing (Playwright MCP)

For AI agents, the Playwright MCP (Model Context Protocol) provides automated browser testing capabilities:

- **Setup**: Ensure the application is running at `http://localhost:5000`
- **Navigation**: Use `mcp_playwright_browser_navigate` to navigate to pages
- **Interactions**: Use tools like `mcp_playwright_browser_click`, `mcp_playwright_browser_type` for user actions
- **Verification**: Use `mcp_playwright_browser_snapshot` to capture page state
- **Console**: Use `mcp_playwright_browser_console_messages` to check for JavaScript errors

**Example workflow:**
1. Navigate to the application
2. Complete setup wizard if needed
3. Navigate to specific features (e.g., `/Admin/Media`)
4. Interact with UI elements
5. Verify console has no errors
6. Capture screenshots or snapshots for validation

### Test Organization

- `test/OrchardCore.Tests/` - Main unit test project
- `test/OrchardCore.Abstractions.Tests/` - Tests for abstractions
- `test/OrchardCore.Tests.Functional/` - Playwright E2E functional tests
- `test/OrchardCore.Tests.Modules/` - Test modules used by tests

## Project Structure

```
OrchardCore/
├── src/
│   ├── OrchardCore/                    # Core framework libraries
│   │   ├── OrchardCore/                # Main framework
│   │   ├── OrchardCore.Abstractions/   # Core interfaces
│   │   └── ...                         # Other abstractions
│   ├── OrchardCore.Modules/            # Built-in modules
│   ├── OrchardCore.Themes/             # Built-in themes
│   ├── OrchardCore.Cms.Web/            # Main CMS web application
│   └── docs/                           # Documentation source
├── test/                               # Test projects
└── .scripts/                           # Build and asset scripts
```

## Available Skills

The following skills are available in `.agents/skills/` for guided workflows:

| Skill | Description | Use When |
|-------|-------------|----------|
| `orchardcore-module-creator` | Create new modules | Adding modules, content parts, fields, handlers |
| `orchardcore-theme-creator` | Create new themes | Adding themes, layouts, frontend assets |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OrchardCMS/OrchardCore](https://github.com/OrchardCMS/OrchardCore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
