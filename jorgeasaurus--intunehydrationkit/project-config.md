---
trigger: always_on
description: Intune Hydration Kit is a PowerShell 7 module that bootstraps Microsoft Intune tenants with best-practice baseline configurations. It uses `Invoke-MgGraphRequest` (only `Microsoft.Graph.Authentication` required) to import policies, groups, compliance baselines, enrollment profiles, conditional access, mobile apps, and device filters. If required Graph scopes are missing, instruct the user to grant the missing permissions and stop the operation.
---

# IntuneHydrationKit - Copilot Instructions

## Project Overview

Intune Hydration Kit is a PowerShell 7 module that bootstraps Microsoft Intune tenants with best-practice baseline configurations. It uses `Invoke-MgGraphRequest` (only `Microsoft.Graph.Authentication` required) to import policies, groups, compliance baselines, enrollment profiles, conditional access, mobile apps, and device filters. If required Graph scopes are missing, instruct the user to grant the missing permissions and stop the operation.

## Build, Test, and Lint

Use the InvokeBuild-based bootstrap script (installs dependencies automatically):

```powershell
# Full CI pipeline: Analyze + Test + Build
./build.ps1 -Task CI

# Run specific tasks
./build.ps1 -Task Analyze          # PSScriptAnalyzer only
./build.ps1 -Task Test             # Pester tests only
./build.ps1 -Task Build            # Build module to build/IntuneHydrationKit/
./build.ps1 -Task Clean            # Remove build artifacts

# Direct commands (module must be imported first for tests)
Import-Module ./IntuneHydrationKit.psd1 -Force
Invoke-Pester -Path ./Tests/Private/Invoke-GraphBatchOperation.Tests.ps1 -Output Detailed
Invoke-ScriptAnalyzer -Path ./Public/Orchestration/Invoke-IntuneHydration.ps1
```

PSScriptAnalyzer excludes `PSUseShouldProcessForStateChangingFunctions` and `PSAvoidUsingConvertToSecureStringWithPlainText` (handled manually).

## High-Level Architecture

### Module Structure
- `IntuneHydrationKit.psm1` - Root module. Defines `$script:` state variables and dot-sources `Public/**/*.ps1` and `Private/**/*.ps1`.
- `Public/` - 19 exported functions (one file per function). Main entry point: `Invoke-IntuneHydration`.
- `Private/` - Internal helpers (batch operations, pagination, template loading, result formatting, auth).
- `Templates/` - JSON templates organized by resource type (OpenIntuneBaseline, Compliance, Enrollment, DynamicGroups, StaticGroups, Filters, ConditionalAccess, AppProtection, MobileApps, Notifications).

### Execution Flow (`Invoke-IntuneHydration`)
High-level flow: authenticate and validate prerequisites, process template-driven imports/deletes by resource type, then generate reports.
Runs 12 sequential steps:
1. Authenticate via `Connect-MgGraph`
2. Pre-flight checks (`Test-IntunePrerequisites`) - Intune license, MDM authority
3. Create/delete dynamic groups (`Invoke-GroupBatchImport`)
3b. Create/delete static groups
4. Import/delete device filters
5. Import/delete OpenIntuneBaseline policies (routed by `@odata.type` or folder name)
6. Import/delete compliance templates
7. Import/delete notification templates
8. Import/delete app protection policies (MAM)
9. Import/delete enrollment profiles (Autopilot, ESP, DEP, Device Prep)
10. Import/delete conditional access policies (always disabled on import)
11. Import/delete mobile apps
12. Generate summary report (`Reports/Hydration-Summary.md` + `.json`)

### State & Configuration
- `$script:HydrationState` - connection status, tenant ID, results (Groups, Policies, Baselines, Profiles, ConditionalAccess, Errors, Warnings)
- `$script:ImportPrefix = '[IHD] '` - prepended to most resource display names; mobile apps and WinGet apps append ` - [IHD]`
- `$script:HydrationMarker` / `$script:HydrationMarkerAlt` - description marker for safe deletion
- Settings file: `settings.json` (validated against `settings.schema.json`)

## Key Conventions

### One File Per Function
Every function lives in its own `.ps1` file named exactly after the function. Public functions are exported in both the `.psd1` manifest and the `.psm1` `$publicFunctions` array. Keep them in sync.

### Graph API Batch Pattern
All imports/deletions that touch Graph use `Invoke-GraphBatchOperation`:
- Batches up to `$script:MaxBatchSize` (default 10) items per request
- Retry with exponential backoff for 429 (throttle), 500+, 503
- POST bodies are sent as raw JSON strings (not hashtables) to avoid PowerShell serialization issues
- DELETE items need `Id`; POST items need `BodyJson`
- Batch responses use string IDs (1-indexed) - always parse with `[int]::TryParse()`

### Idempotency & Dual-Lookup
Importers check for both prefixed (`[IHD] Name`) and legacy unprefixed (`Name`) display names before creating, to prevent duplicates across runs.

### Template-Scoped Deletion
Delete operations only remove objects that have BOTH the hydration marker in their description AND a matching template name. If only one condition is met, the object is not deleted. This prevents accidental deletion of manually created resources.

### Baseline Import Routing (`Import-IntuneBaseline`)
Uses two routing strategies:
- **IntuneManagement path** (`$intuneManagementFolders`): Routes by `@odata.type` via `$odataTypeToEndpoint` map. Used for `IntuneManagement/` and `AppProtection/` subfolders.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jorgeasaurus/IntuneHydrationKit](https://github.com/jorgeasaurus/IntuneHydrationKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
