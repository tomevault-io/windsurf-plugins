---
trigger: always_on
description: This file gives AI coding agents the repository-specific context needed to make correct, focused changes. It applies to the whole repository unless a nested `AGENTS.md` appears in a subdirectory.
---

# AGENTS.md - WeihanLi.Common

This file gives AI coding agents the repository-specific context needed to make correct, focused changes. It applies to the whole repository unless a nested `AGENTS.md` appears in a subdirectory.

## Project Overview

`WeihanLi.Common` is a production-ready .NET utility library that bundles helpers, extensions, middleware, and application primitives for .NET development. It covers AOT-friendly core utilities, dependency injection, Fluent Aspects AOP, eventing, logging, data access helpers, OTP utilities, templating, hosting helpers, and more.

Four NuGet packages are published from this repository:

| Package | Description |
|---|---|
| `WeihanLi.Core` | AOT-compatible core extensions, helpers, models, compression primitives, OTP utilities, and dependency-light APIs |
| `WeihanLi.Common` | Full utility package built on `WeihanLi.Core`, including DI, AOP, logging, data access, HTTP, JSON, configuration, templating, and eventing |
| `WeihanLi.Common.Logging.Serilog` | Serilog integration layer for `WeihanLi.Common.Logging` |
| `WeihanLi.Extensions.Hosting` | Hosting helpers for background services, console apps, and DI bootstrapping |

## Repository Map

```text
src/
  WeihanLi.Core/                    AOT-friendly core package; root namespace remains WeihanLi.Common
  WeihanLi.Common/                  Full utility package referencing WeihanLi.Core
  WeihanLi.Common.Logging.Serilog/  Serilog integration package
  WeihanLi.Extensions.Hosting/      Hosting extensions package
test/
  WeihanLi.Common.Test/             xUnit v3 tests on Microsoft Testing Platform
samples/
  AspNetCoreSample/                 ASP.NET Core examples
  DotNetCoreSample/                 Console examples
perf/
  WeihanLi.Common.Benchmark/        BenchmarkDotNet benchmarks
docs/                               DocFX documentation
build/                              Versioning and signing MSBuild props
```

Important root files:

| File | Purpose |
|---|---|
| `WeihanLi.Common.slnx` | Solution file |
| `build.cs` | C# build script used by CI |
| `Directory.Build.props` | Shared MSBuild properties; sets `LatestTargetFramework` to `net10.0` |
| `src/Directory.Build.props` | Source-package MSBuild properties, docs, packaging, signing, Source Link |
| `Directory.Packages.props` | Central package management and NuGet audit configuration |
| `.github/workflows/default.yml` | GitHub Actions CI |
| `azure-pipelines.yml` | Azure Pipelines CI |

There is currently no `global.json`; use the SDK versions required by the project and CI instead of assuming an SDK pin.

## Build And Test

Prerequisites:

- Install .NET SDK `10.0.x` and .NET SDK `11.0.x` preview when building all source projects, because `WeihanLi.Core` and `WeihanLi.Common` target `net11.0`.
- CI installs both SDKs. Azure Pipelines uses `includePreviewVersions: true` for .NET 11.

Common commands:

```bash
# Build the solution
dotnet build

# Run all tests
dotnet test

# Full CI build: build, test, and pack according to build.cs
dotnet build.cs

# Verify formatting
dotnet format --verify-no-changes

# Apply formatting
dotnet format
```

Set `DISABLE_GITHUB_ACTIONS_TEST_LOGGER=true` to opt out of the GitHub Actions test logger when running tests locally.

Before submitting a change, run the narrowest relevant test first, then run these required checks when practical:

```bash
dotnet build
dotnet test
dotnet format --verify-no-changes
```

## Target Frameworks

| Project | Target frameworks |
|---|---|
| `src/WeihanLi.Core` | `netstandard2.0`, `net8.0`, `net10.0`, `net11.0` |
| `src/WeihanLi.Common` | `netstandard2.0`, `net8.0`, `net10.0`, `net11.0` |
| `src/WeihanLi.Common.Logging.Serilog` | `netstandard2.0`, `net10.0` |
| `src/WeihanLi.Extensions.Hosting` | `net10.0` |
| `test/WeihanLi.Common.Test` | `$(LatestTargetFramework)` (`net10.0`) |

Use conditional compilation for framework-specific behavior, for example `#if NET8_0_OR_GREATER`, `#if NET10_0_OR_GREATER`, or explicit `netstandard2.0` conditions. Be careful with `Reflection.Emit` and APIs missing from `netstandard2.0`.

## Key Namespaces And Components

| Namespace | Purpose |
|---|---|
| `WeihanLi.Common` | Core utilities such as `Guard`, `CacheUtil`, and `DependencyResolver` |
| `WeihanLi.Common.Abstractions` | Shared primitives and property bags |
| `WeihanLi.Common.Aspect` | Fluent Aspects AOP, dynamic proxies, interceptors, and invocation pipeline |
| `WeihanLi.Common.Compressor` | Compression helpers from `WeihanLi.Core` |
| `WeihanLi.Common.Data` | ADO.NET extensions, repository helpers, entity mapping, SQL expression parsers |
| `WeihanLi.Common.DependencyInjection` | Lightweight DI container, service definitions, modules, decorators |
| `WeihanLi.Common.Event` | `EventBus`, `EventQueue`, `EventStore`, publish/subscribe helpers |
| `WeihanLi.Common.Helpers` | `ApplicationHelper`, `TotpHelper`, `CommandExecutor`, `ConsoleHelper`, cron and pipeline helpers |
| `WeihanLi.Common.Http` | HTTP client utilities and mock handlers |
| `WeihanLi.Common.Json` | JSON converters and serialization helpers |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WeihanLi/WeihanLi.Common](https://github.com/WeihanLi/WeihanLi.Common) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
