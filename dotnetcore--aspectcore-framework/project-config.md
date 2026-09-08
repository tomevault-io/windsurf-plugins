---
trigger: always_on
description: Project-level AI context for the AspectCore-Framework repository. Generated from the current code tree (version `3.0.0-rc.1`). Keep this file lean; link to external docs instead of inlining them.
---

# AGENTS.md — AspectCore-Framework

Project-level AI context for the AspectCore-Framework repository. Generated from the current code tree (version `3.0.0-rc.1`). Keep this file lean; link to external docs instead of inlining them.

---

## 1. Project Overview

**AspectCore-Framework** is an Aspect-Oriented Programming (AOP) framework for .NET. It weaves interceptors into service methods through two equivalent proxy engines that share one contract (`AspectCore.Abstractions`):

- **DynamicProxy** (runtime, IL emit via `System.Reflection.Emit`) — lives in `AspectCore.Core`.
- **Source Generator** (compile-time, Roslyn `IIncrementalGenerator`) — lives in `AspectCore.SourceGenerator`.

**Tech stack (concrete):**
- **.NET target frameworks (libraries):** `net10.0;net9.0;net8.0;net6.0` (SourceGenerator is `netstandard2.0` only, as required by Roslyn).
- **.NET target frameworks (tests):** `net10.0;net9.0;net8.0;net6.0`.
- **C# language version:** `13.0` for `src/` (set in `build/common.props`); `13.0` for tests.
- **Test framework:** xUnit `2.9.2` + `Microsoft.NET.Test.Sdk 17.12.0`.
- **Coverage:** `coverlet.msbuild 6.0.2` (Cobertura), thresholds enforced in CI (unit 95%, E2E 80%).
- **DI integrations:** MsDI, Autofac `[7.0.0, 8.0.0)`, Castle.Windsor `6.0.0`, LightInject `6.6.4`, plus Generic Host and ASP.NET Core adapters.
- **Benchmarks:** BenchmarkDotNet `0.14.0`.
- **Version source of truth:** `build/version.props` (`VersionMajor=3`, `VersionMinor=0`, `VersionPatch=0`, `VersionQuality=rc.1`).

---

## 2. Project Structure Map

| Directory | Purpose | Local Documentation |
|-----------|---------|---------------------|
| `src/AspectCore.Abstractions/` | Pure contracts: interfaces, attributes, enums. No implementation. Namespaces `AspectCore.DynamicProxy`, `AspectCore.Configuration`, `AspectCore.DependencyInjection`. | – |
| `src/AspectCore.Core/` | Runtime DynamicProxy engine (IL emit), built-in IoC container (`ServiceContext`/`ServiceResolver`), interceptor pipeline, configuration. `AllowUnsafeBlocks=true`. | – |
| `src/AspectCore.Extensions.Reflection/` | Standalone high-performance reflection library. **No AspectCore project references**; consumed by Core. | – |
| `src/AspectCore.SourceGenerator/` | Roslyn compile-time proxy generator (`AspectCoreProxyGenerator`). `IsRoslynComponent=true`, `OutputItemType=Analyzer`. No project references. | – |
| `src/AspectCore.Extensions.DependencyInjection/` | Microsoft.Extensions.DependencyInjection (MsDI) adapter. | – |
| `src/AspectCore.Extensions.Autofac/` | Autofac adapter. | – |
| `src/AspectCore.Extensions.Windsor/` | Castle.Windsor adapter. | – |
| `src/AspectCore.Extensions.LightInject/` | LightInject adapter. | – |
| `src/AspectCore.Extensions.Hosting/` | Generic Host integration. | – |
| `src/AspectCore.Extensions.AspNetCore/` | ASP.NET Core web integration (`FrameworkReference Microsoft.AspNetCore.App`). | – |
| `src/AspectCore.Extensions.AspectScope/` | ScopedContext / aspect scope extension. | – |
| `src/AspectCore.Extensions.Configuration/` | Configuration injection via `Microsoft.Extensions.Configuration`. | – |
| `src/AspectCore.Extensions.DataAnnotations/` | DataAnnotations-based validation extension. | – |
| `src/AspectCore.Extensions.DataValidation/` | Data validation extension. | – |
| `src/AspectCore.Extensions.CastleCompat/` | Castle DynamicProxy compatibility shim for gradual migration to AspectCore. Depends on `Castle.Core`. Targets `net10.0;net9.0;net8.0`. | – |
| `tests/` | 10 xUnit test projects + `AspectCore.NativeAot.E2E` (a `PublishAot` executable, not xUnit). `tests/Directory.Build.props` injects `coverlet.msbuild`. | – |
| `sample/` | 4 runnable sample projects (DI console, AspectScope, Autofac, DataAnnotations). | – |
| `benchmark/` `benchmarks/` | BenchmarkDotNet projects. | – |
| `docs/` | Architecture, guide, getting-started, development, testing docs (bilingual; `docs/en/` for English). | `docs/README.md` |
| `build/` | `common.props`, `version.props`, `sign.props`, `aspectcore.snk`. | – |
| `.github/workflows/` | `build-ci.yml`, `build-pr-ci.yml`, `release.yml`. | – |

**Dependency direction (acyclic, bottom-up):**
`Abstractions` + `Extensions.Reflection` ◄── `Core` ◄── all Extensions. `SourceGenerator` is independent (no project refs; generated code references Core/Abstractions at runtime).

---

## 3. Build & Development Commands

> No `global.json` exists — SDK is not pinned. CI installs `6.0.x / 8.0.x / 9.0.x / 10.0.x`. Locally you need an SDK that can build the target frameworks you care about.

```bash
# Whole solution
dotnet build AspectCore-Framework.sln --configuration Release

# Per-project (matches CI behavior)
for project in $(find ./src -name "*.csproj"); do
  dotnet build --configuration Release "$project"
done

# Build with explicit version (CI release flow)
dotnet build --configuration Release ./src/AspectCore.Core/AspectCore.Core.csproj -p:Version=3.0.0-rc.1

# Format check (PR CI gate — currently warns, does not fail)
dotnet format AspectCore-Framework.sln --verify-no-changes

# Auto-format locally before pushing
dotnet format AspectCore-Framework.sln
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dotnetcore/AspectCore-Framework](https://github.com/dotnetcore/AspectCore-Framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
