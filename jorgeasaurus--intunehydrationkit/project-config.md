---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

Intune Hydration Kit is a PowerShell module that bootstraps Microsoft Intune tenants with best-practice defaults. It hydrates tenants with:
- OpenIntuneBaseline policies (99 bundled JSON templates - no external download required)
- CIS Baselines (728 bundled benchmark-derived policies)
- Compliance Baseline Pack (10 multi-platform policies)
- Enrollment Profiles (Autopilot, Self-Deploy, ESP, macOS DEP, Device Preparation)
- Dynamic Group Suite (50 groups) and Static Groups (5 groups)
- Device Filters (24 platform, manufacturer, and VM filters)
- App Protection Policies (8 MAM policies + 2 BYOD baseline policies)
- Notification Templates
- Mobile Apps (8 legacy templates plus 28 WinGet Win32 app templates)
- Conditional Access Starter Pack (21 policies, all disabled by default)

Uses Microsoft Graph API via `Invoke-MgGraphRequest` (only `Microsoft.Graph.Authentication` module required). Most resources are prefixed with `[IHD] ` for instant identification; mobile apps and WinGet apps append ` - [IHD]`. Created objects are tagged with a hydration kit marker for safe deletion.

## Build and Test Commands

### InvokeBuild System (Primary)

```powershell
# Bootstrap build environment and run default tasks (Analyze + Test + Build)
./build.ps1

# Run PSScriptAnalyzer linting only
./build.ps1 -Task Analyze

# Run Pester tests only
./build.ps1 -Task Test

# Build module to build/IntuneHydrationKit/
./build.ps1 -Task Build

# CI task - full validation without publishing (Analyze + Test + Build)
./build.ps1 -Task CI

# Clean build artifacts
./build.ps1 -Task Clean
```

### Direct Commands

```powershell
# Run all Pester tests
Invoke-Pester -Path ./Tests -Output Detailed

# Run a single test file
Invoke-Pester -Path ./Tests/Public/Connect-IntuneHydration.Tests.ps1 -Output Detailed

# Lint with ScriptAnalyzer
Invoke-ScriptAnalyzer -Path . -Recurse

# Import the module locally for development
Import-Module ./IntuneHydrationKit.psd1 -Force
```

### Running the Module

```powershell
# Setup: Copy and configure settings (schema at settings.schema.json)
Copy-Item settings.example.json settings.json
# Edit settings.json with your tenant details

# Dry-run mode (validates without writing to Graph)
pwsh ./Invoke-IntuneHydration.ps1 -SettingsPath ./settings.json -WhatIf

# Live run with force update
pwsh ./Invoke-IntuneHydration.ps1 -SettingsPath ./settings.json -Force

# Parameter-based invocation (no settings file)
Invoke-IntuneHydration -TenantId "guid" -Interactive -Create -All -WhatIf

# Delete all hydration kit resources then recreate
Invoke-IntuneHydration -Interactive -Delete -TenantId "guid" -All -Force
Invoke-IntuneHydration -Interactive -Create -TenantId "guid" -All
```

## Architecture

### Entry Point

- `Invoke-IntuneHydration.ps1` - Wrapper script (backward compatibility for cloned repo)
- `Public/Orchestration/Invoke-IntuneHydration.ps1` - Main orchestrator function (used when installed from PSGallery)

### Module Structure

- `IntuneHydrationKit.psd1` / `IntuneHydrationKit.psm1` - Module manifest and loader (dot-sources Public/Private)
- `Public/` - 19 exported public functions (one file per function)
- `Private/` - 95 internal helper functions

### Module-Level State

The module maintains state in script-scoped variables (defined in `IntuneHydrationKit.psm1`):
- `$script:HydrationState` - Tracks connection status, tenant ID, and results (Groups, Policies, Baselines, Profiles, ConditionalAccess, Errors, Warnings)
- `$script:LogPath`, `$script:CurrentLogFile`, `$script:VerboseLogging` - Logging state
- `$script:GraphEnvironment`, `$script:GraphEndpoint` - Graph API configuration
- `$script:ImportPrefix` - Resource naming prefix, defaults to `'[IHD] '`
- `$script:MaxBatchSize` - Batch API item limit per request, defaults to `10` (Graph max is 20)
- `$script:TemplatesPath` - Path to Templates directory

### Templates Directory

All JSON templates for import operations:

- `Templates/OpenIntuneBaseline/` - Bundled baseline security policies (OS/PolicyType/policy.json structure)
- `Templates/Compliance/` - Platform compliance policies (Windows, iOS, macOS, Android, Linux)
- `Templates/Enrollment/` - Autopilot, ESP, macOS DEP, and Device Preparation profiles
- `Templates/DynamicGroups/` - OS, Manufacturer, and Autopilot group definitions
- `Templates/StaticGroups/` - Assigned groups (Update Ring Pilot, UAT)
- `Templates/Filters/` - Device filter definitions
- `Templates/ConditionalAccess/` - CA starter pack (forced disabled on import)
- `Templates/AppProtection/` - Android and iOS MAM policy templates
- `Templates/MobileApps/` - macOS, Microsoft Store/M365 fallback, and Windows WinGet app templates
- WinGet app imports are limited to bundled templates. New apps should be requested by issue or added by PR with template metadata.
- `Templates/Notifications/` - Notification message templates

### Build System

- `build.ps1` - Bootstrap script that installs InvokeBuild and runs tasks
- `IntuneHydrationKit.build.ps1` - InvokeBuild task definitions (Analyze, Test, Build)
- `scripts/New-MobileAppTemplate.ps1` - Generate mobile app JSON templates

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jorgeasaurus/IntuneHydrationKit](https://github.com/jorgeasaurus/IntuneHydrationKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
