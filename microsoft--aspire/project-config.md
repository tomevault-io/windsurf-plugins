---
trigger: always_on
description: Instructions for GitHub Copilot and other AI coding agents working with the Aspire repository.
---

# Agent Instructions

Instructions for GitHub Copilot and other AI coding agents working with the Aspire repository.

## Repository Overview

**Aspire** provides tools, templates, and packages for building observable, production-ready distributed applications. At its core is an app model that defines services, resources, and connections in a code-first approach.

### Key Components
- **Aspire.Hosting**: Application host orchestration and resource management
- **Aspire.Dashboard**: Web-based dashboard for monitoring and debugging
- **Service Discovery**: Infrastructure for service-to-service communication
- **Integrations**: 40+ packages for databases (SQL Server, PostgreSQL, Redis, MongoDB), message queues (RabbitMQ, Kafka), cloud services (Azure), and more
- **CLI Tools**: Command-line interface for project creation and management
- **Project Templates**: Starter templates for new Aspire applications

### Technology Stack
- .NET 10.0
- C# 13 preview features
- xUnit SDK v3 with Microsoft.Testing.Platform for testing
- Microsoft.DotNet.Arcade.Sdk for build infrastructure
- Native AOT compilation for CLI tools
- Multi-platform support (Windows, Linux, macOS, containers)

## General

* Make only high confidence suggestions when reviewing code changes.
* Always use the latest version C#, currently C# 13 features.
* Never change global.json unless explicitly asked to.
* Never change package.json or package-lock.json files unless explicitly asked to.
* Never change NuGet.config files unless explicitly asked to.
* Do not use cryptographic hashes such as SHA-256 when the hash is not security-related. Prefer `System.IO.Hashing.XxHash3` when you need a stable non-cryptographic hash.
* When code needs a temporary directory, prefer the repository temp directory abstractions first (for example `IFileSystemService.TempDirectory` / `ITempFileSystemService`) and otherwise use `Directory.CreateTempSubdirectory()` instead of `Path.Combine(Path.GetTempPath(), Path.GetRandomFileName())`; if you need a temporary file path, place it under a securely created temp directory.
* Don't update files under `*/api/*.cs` (e.g. src/Aspire.Hosting/api/Aspire.Hosting.cs) as they are generated.
* Do not make new parameters optional just to avoid updating call sites. A parameter should only be optional when it has a sensible semantic default and the API is frequently used (where call-site brevity outweighs explicitness). If a parameter is logically required, make it required and update all call sites.

## Code Review Instructions

### API Files and Public API Surface

The API files located in `*/api/*.cs` (e.g., `src/Aspire.Hosting/api/Aspire.Hosting.cs`) track the public API surface that has already been shipped in the latest release. These files are auto-generated and serve as a baseline for API compatibility checks.

When reviewing pull requests:

* **Do not comment when new public API is introduced and the API files are not regenerated**. This is expected behavior during active development between releases.
* New public APIs should be reviewed for design, naming, and functionality, but the absence of API file updates during PR development is normal.
* API files are regenerated as part of the release process when we ship a new version, not during individual PRs.
* Only flag API file concerns if:
  - API files are manually edited (they should never be manually modified)
  - There are breaking changes to existing APIs without proper justification
  - The PR explicitly claims to update API compatibility but doesn't regenerate the files

### NuGet Feed Configuration

The NuGet.config file defines approved package sources for the internal build. External package feeds can break the internal build pipeline.

When reviewing pull requests:

* **Flag any changes to NuGet.config that add package sources not from these approved domains:**
  - `https://pkgs.dev.azure.com/dnceng`
  - `https://dnceng.pkgs.visualstudio.com/public`
* **Flag any additions of external NuGet feeds** such as:
  - `https://api.nuget.org/v3/index.json` (nuget.org)
  - Any other public or third-party package sources
* If a PR adds an external feed, request that:
  - The packages be mirrored to an approved internal feed, or
  - Use existing internal feeds that already mirror public packages (like dotnet-public, dotnet-eng)
* The wildcard pattern mappings (`<package pattern="*" />`) in dotnet-public and dotnet-eng feeds typically provide access to commonly-used public packages

## Formatting

* Apply code-formatting style defined in `.editorconfig`.
* Prefer file-scoped namespace declarations and single-line using directives.
* Insert a newline before the opening curly brace of any code block (e.g., after `if`, `for`, `while`, `foreach`, `using`, `try`, etc.).
* Ensure that the final return statement of a method is on its own line.
* Use pattern matching and switch expressions wherever possible.
* Use `nameof` instead of string literals when referring to member names.
* Place private class declarations at the bottom of the file.

### Code comments

* Err on the side of over-commenting code when the reasoning is not obvious. Comments should explain **WHY** code is written a particular way; the **WHY** is the most important part.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/aspire](https://github.com/microsoft/aspire) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
