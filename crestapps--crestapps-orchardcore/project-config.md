---
trigger: always_on
description: **ALWAYS reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**
---

# CrestApps.OrchardCore Development Instructions

**ALWAYS reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**

## Project Overview

CrestApps.OrchardCore is a collection of open-source modules for **Orchard Core CMS**, a modular application framework built on **ASP.NET Core/.NET 10**. The repository contains AI modules, omnichannel communication, user management enhancements, content access control, and other CMS extensions.

**License**: MIT  
**Target Framework**: .NET 10.0 (net10.0)  
**Architecture**: Modular, multi-tenant application framework

## Working Effectively

### Prerequisites and Environment Setup

Install .NET 10.0 SDK first:
```bash
# Add Microsoft package repository (Ubuntu/Debian)
wget https://packages.microsoft.com/config/ubuntu/22.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
rm packages-microsoft-prod.deb

# Install .NET 10.0 SDK - TAKES 1-2 MINUTES
sudo apt-get update
sudo apt-get install -y dotnet-sdk-10.0

# Verify installation
dotnet --version  # Should show 10.0.x
```

### Build Process

**CRITICAL BUILD LIMITATION**: The .NET build requires access to Orchard Core 3.0 preview packages from `https://nuget.cloudsmith.io/orchardcore/preview/v3/index.json`. In environments with restricted network access (like CI runners or sandboxed environments), the build will fail with network connectivity errors.

#### Asset Build (Always Works)
```bash
# Install npm dependencies - TAKES 2-3 MINUTES, NEVER CANCEL
npm install

# Build frontend assets - TAKES 4 SECONDS
npm run rebuild
# OR
gulp rebuild
```

#### .NET Solution Build (Network Dependent)
```bash
# Full solution build - ONLY WORKS WITH NETWORK ACCESS TO CLOUDSMITH
# TAKES 5-10 MINUTES when successful, NEVER CANCEL
dotnet build -c Release -warnaserror /p:TreatWarningsAsErrors=true /p:RunAnalyzers=true /p:NuGetAudit=false
```

**Important for Copilot skills**: In a fresh environment, build the project early and then check for newly generated skills under `.agents\skills`. One of the packages can create `.agents\skills` on the first build, and those generated skills may contain important, up-to-date capabilities that are not visible before the build runs.

**If build fails with NU1301 errors** about `nuget.cloudsmith.io`, this is expected in restricted environments. Document this limitation rather than attempting workarounds.

#### Unit Tests (Requires Successful Build)
```bash
# Run unit tests - TAKES 2-5 MINUTES, NEVER CANCEL
dotnet test -c Release --no-build ./tests/CrestApps.OrchardCore.Tests/CrestApps.OrchardCore.Tests.csproj
```

### Running the Application

**Web Application**: The main application is in `src/Startup/CrestApps.OrchardCore.Cms.Web/`

```bash
# Run the CMS web application (requires successful build)
cd src/Startup/CrestApps.OrchardCore.Cms.Web
dotnet run

# Application runs on: http://localhost:5000
# Admin setup occurs on first run
```

**Aspire Orchestration**: For full-stack local development with Ollama, Elasticsearch, and Redis, use the Aspire AppHost:

```bash
cd src/Startup/CrestApps.Aspire.AppHost
dotnet run
```

### Code Guidelines

* Follow `.editorconfig` at all times.
* Prefer constructor injection.
* Do not add `ArgumentNullException.ThrowIf...` guards in constructors.

#### Null Handling

* Add null guards in public implementation methods when a non-nullable input is required and the method does not intentionally support `null`.
* Skip null guards for nullable or intentionally null-tolerant parameters.
* After the final null-check or argument-validation line in a method, add a blank line before the next statement (even if it is the only guard).

#### Formatting & Layout

* Never use more than one consecutive blank line.
* Add a blank line before a `return` statement unless it is the first statement in a `{ ... }` block.
* Add a blank line before and after `if`, `switch`, and loop blocks unless the block is immediately preceded by `{`.
* Do not add a blank line between a control statement (`if`/`else`/`switch`/loop) and its opening `{`.
* Do not leave extra blank lines between consecutive closing braces.
* Add a blank line after `};` and after multi-line object initializers unless returned inline.
* When an object initializer spans multiple lines, place each property assignment on its own line.
* Format conditional (`?:`) operators across multiple lines with the condition, `?`, and `:` each on their own properly indented lines.
* Always keep exactly one trailing newline at the end of each file.
* In `.cshtml` files, never introduce stray carriage returns (`^M`) or CRCRLF line endings; keep view files normalized so they do not render with artificial blank lines.

#### `#pragma` Rules

* Do not add a blank line immediately after `#pragma warning disable`.
* Do not add a blank line immediately before `#pragma warning restore`.
* Do not add a blank line after `#pragma warning restore` if followed by `{`.
* Add a blank line before a `#pragma warning disable` block when it starts a new member after a `}`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CrestApps/CrestApps.OrchardCore](https://github.com/CrestApps/CrestApps.OrchardCore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
