---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Doc Formatter & Hyperlink Fixer — Architecture & Implementation Outline | Project Name = "Doc_Medic"

## Development Commands

Since this project uses .NET 9, standard .NET CLI commands apply:
- **Build solution**: `dotnet build src/App.sln --configuration Release`
- **Run tests**: `dotnet test src/App.sln --configuration Release`
- **Run specific test**: `dotnet test --filter "FullyQualifiedName~TestClassName.TestMethodName"`
- **Restore packages**: `dotnet restore src/App.sln`
- **Clean**: `dotnet clean src/App.sln`
- **Local Velopack build**: `.\build.ps1 -Version "1.0.0" -Configuration "Release"`

## Implementation Status

✅ **COMPLETED**
1. Complete project structure with all assemblies (App.UI, App.Domain, App.Core, App.Infrastructure, App.Services, App.Tests)
2. All core interfaces implemented (App.Core)
3. Full OpenXML document processing pipeline (App.Infrastructure)
4. WPF UI with Prism MVVM and Material Design theming (App.UI)
5. Real service integrations replacing all mock implementations
6. Velopack deployment configuration with GitHub Actions CI/CD
7. Settings and secrets management with DPAPI encryption
8. Comprehensive logging with Serilog
9. HTTP client with Polly resilience policies

## Implementation Priority (Historical)
1. ✅ Start with the explicit project structure (section 2)
2. ✅ Implement interfaces first (from section 9) 
3. ✅ Follow the pseudocode in section 21
4. ✅ Use the OpenXML helpers in section 8 as implementation guides
5. ✅ The regex patterns and processing logic are production-ready

> Purpose: Modern desktop app to batch‑format .docx files and repair specific hyperlinks via a Power Automate API. This document is a coding brief for generating the full solution with Claude CLI.

---

## 0) Goals & Non‑Goals

**Goals**

* Batch select many `.docx` files and apply user‑selected transforms reliably and fast.
* Fix **only** hyperlinks that match defined patterns (CMS, TSRC, or `?docid=`) using a Power Automate API.
* Standardize styles (Normal, Heading 1, Heading 2, Hyperlink) to the specified rules.
* Optional cleanups: collapse multiple spaces, normalize “Top of Document” links, center images.
* Seamless in‑app updates from GitHub Releases.
* Modern, sleek, responsive WPF UI with theming, persistent settings, and verbose logging.

**Non‑Goals**

* Editing formats other than `.docx`.
* General link validation beyond the specified patterns.

---

## 1) Platform & Tech Stack

* **Runtime**: .NET 9 (Windows)
* **UI**: WPF + Prism (MVVM, modular), modern theme via **MaterialDesignInXaml**.
* **Document IO**: Open XML SDK (`DocumentFormat.OpenXml`). No Word COM automation.
* **Dependency Injection**: Prism + DryIoc.
* **Updater**: Velopack
* **HTTP**: `HttpClient` w/ Polly for retry & timeout.
* **Config**: JSON settings at `%AppData%/Company/App/settings.json`, secrets (there is no API key for Power Automate HTTP Request) encrypted via DPAPI.
* **Logging**: Serilog (rolling file logs + optional console sink in debug).
* **Testing**: xUnit (regex/unit), approval tests for XML transforms.

---

## 2) Solution Structure

```
repo-root/
  src/
    App.UI/                    # ✅ WPF shell (Prism), views, viewmodels, navigation
    App.Domain/                # ✅ Domain models, enums, constants, value objects
    App.Core/                  # ✅ Abstractions: services, ports, pipeline stages
    App.Infrastructure/        # ✅ OpenXML impl, HTTP client, storage, updater, logging
    App.Services/              # ✅ Orchestration services (document pipeline, styles, links)
    App.CLI/                   # (Optional) headless batch runner using same services
  tests/
    App.Tests/                 # ✅ Unit + integration tests
  .github/
    workflows/
      release.yml              # ✅ GH Actions to build, sign, and publish releases
  tools/
    schemas/                   # JSON Schemas for API payloads/responses (if needed)
  velopack.json                # ✅ Velopack package configuration
  deployment.json              # ✅ Production deployment settings  
  build.ps1                    # ✅ Local build and packaging script
  Directory.Packages.props     # ✅ Centralized package management
  src/App.sln                  # ✅ Visual Studio solution file
  CLAUDE.md                    # this file
```

**Key assembly boundaries**

* `App.Core`: ✅ Interfaces + contracts (stable). No external deps.
* `App.Domain`: ✅ Pure C# types (POCOs), regexes, constants.
* `App.Infrastructure`: ✅ All side‑effects (IO, network, OpenXML, updater, DPAPI, file system).
* `App.Services`: ✅ Application logic, pipelines, composition over `App.Core` ports.
* `App.UI`: ✅ MVVM only; no OpenXML or HTTP directly.

---

## 3) Public Features (User Flows)

1. **Select files**: drag‑drop or file picker -> shows a table with per‑file status, size, last modified.
2. **Options panel** (toggles):

   * Replace double spaces+ with single.
   * Fix “Top of Document” links (internal anchor, right‑aligned, styled).
   * Standardize styles (Normal, Heading 1, Heading 2, Hyperlink).
   * Center all images.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ItMeDiaTech) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
