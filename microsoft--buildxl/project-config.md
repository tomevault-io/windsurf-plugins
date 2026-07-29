---
trigger: always_on
description: Use this skill when the user asks to update .NET runtime packages (e.g., `Microsoft.NETCore.App.Runtime.*`, ASP.NET Core, or related System.* packages).
---

# Updating .NET Runtime Dependencies

Use this skill when the user asks to update .NET runtime packages (e.g., `Microsoft.NETCore.App.Runtime.*`, ASP.NET Core, or related System.* packages).

## Overview

BuildXL pins .NET runtime package versions across several files. An update requires bumping version strings **and** assembly binding redirect versions in lockstep. Forgetting the binding redirects causes `FileLoadException` at test time on .NET Framework (net472) targets.

## Step 1: Determine the latest stable versions

Query NuGet for the latest stable (non-preview) versions of each major .NET release used by the repo:

- **8.0.x**: `https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.win-x64/index.json`
- **9.0.x**: same URL — look for the highest `9.0.*` entry without `-preview`/`-rc`
- **10.0.x**: same URL — look for the highest `10.0.*` entry without `-preview`/`-rc`

## Step 2: Update NuGet package versions

Update these files, replacing the old version strings with the new ones:

### `config.nuget.dotnetcore.dsc`
- `core80Version` constant (e.g., `"8.0.26"` → `"8.0.27"`)
- `core90Version` constant (e.g., `"9.0.15"` → `"9.0.16"`)
- `core100Version` constant (e.g., `"10.0.8"` → `"10.0.9"`)
- Hardcoded version strings for `System.Formats.Asn1`, `System.Text.Json`, and `System.Collections.Immutable`
- Update the comment referencing the latest HostResolver/HostPolicy 8.0.x version

### `config.nuget.aspNetCore.dsc`
- `aspVersion`, `asp8RefVersion`, `asp8RuntimeVersion`, `asp9RefVersion`, `asp9RuntimeVersion`, `asp10RefVersion`, `asp10RuntimeVersion`

### `config.dsc`
- All packages with hardcoded versions matching the old version (e.g., `System.Diagnostics.DiagnosticSource`, `System.Reflection.Metadata`, `System.Threading.Tasks.Dataflow`, `Microsoft.Bcl.AsyncInterfaces`, `System.Threading.Channels`, `System.IO.Hashing`, `System.IO.Pipelines`, `System.Text.Encodings.Web`, `System.Security.Cryptography.Xml`, `System.Security.Cryptography.Pkcs`, `Microsoft.Bcl.Cryptography`, `System.Reflection.MetadataLoadContext`, `System.Resources.Extensions`, `System.CodeDom`, `System.Text.Encoding.CodePages`, `Microsoft.Extensions.Logging.Abstractions`, `System.Formats.Nrbf`)
- **Do NOT change** packages with the `ForVBCS` alias — those stay at their pinned older versions intentionally.

### `Public/Sdk/Public/Managed/Frameworks/net8/net8.0.dsc`
- `runtimeConfigVersion` (e.g., `"8.0.26"` → `"8.0.27"`)

### `Public/Sdk/Public/Managed/Frameworks/net9/net9.0.dsc`
- `runtimeConfigVersion` (e.g., `"9.0.15"` → `"9.0.16"`)

### `Public/Sdk/Public/Managed/Frameworks/net10/net10.0.dsc`
- `runtimeConfigVersion` (e.g., `"10.0.8"` → `"10.0.9"`)

### `cg/nuget/cgmanifest.json`
- Bulk-replace all occurrences of the old 8.0.x, 9.0.x, and 10.0.x version strings with the new ones. This file has many entries (70+); use a bulk find-and-replace.

## Step 3: Update assembly binding redirects

.NET NuGet package version `X.Y.Z` maps to assembly version `X.Y.0.Z`. For example, NuGet `9.0.15` → assembly `9.0.0.15`, NuGet `9.0.16` → assembly `9.0.0.16`.

Update `oldVersion` and `newVersion` in all binding redirect entries across these files:

### `Public/Src/FrontEnd/UnitTests/MsBuild/msbuild.exe.config`
This is the **custom MSBuild config for net472 tests**. It contains XML `<bindingRedirect>` entries for many assemblies. Bulk-replace the old assembly version (e.g., `9.0.0.15`) with the new one (e.g., `9.0.0.16`) in both `oldVersion` and `newVersion` attributes.

### `Public/Sdk/SelfHost/BuildXL/BuildXLSdk.dsc`
- `bxlBindingRedirects()` function — update `Microsoft.Bcl.AsyncInterfaces` redirect

### `Public/Sdk/SelfHost/BuildXL/Testing/XUnitV3/xunitv3framework.dsc`
- Update redirects for `System.Collections.Immutable` and `Microsoft.Bcl.AsyncInterfaces`

### `Public/Src/Tools/Tool.MsBuildGraphBuilder/Tool.MsBuildGraphBuilder.dsc`
- Update `Microsoft.Bcl.AsyncInterfaces` redirect

### `Public/Src/Tools/UnitTests/MsBuildGraphBuilder/Test.Tool.MsBuildGraphBuilder.dsc`
- Update `System.Text.Json` redirect

## Step 4: Update Download resolver runtime URLs and hashes

The `config.dsc` file contains Download resolver entries for standalone .NET runtimes (used for Linux builds and certain test scenarios). Update the URLs and content hashes for all 9 entries (3 platforms × 3 versions):

### Update URLs
Change version strings in the URLs from old to new (e.g., `10.0.8` → `10.0.9`) for:
- `DotNet-Runtime.win-x64.10.0`, `DotNet-Runtime.osx-x64.10.0`, `DotNet-Runtime.linux-x64.10.0`
- `DotNet-Runtime.win-x64.9.0`, `DotNet-Runtime.osx-x64.9.0`, `DotNet-Runtime.linux-x64.9.0`
- `DotNet-Runtime.win-x64.8.0`, `DotNet-Runtime.osx-x64.8.0`, `DotNet-Runtime.linux-x64.8.0`

Also update the comment line above each group (e.g., `// DotNet Core Runtime 10.0.8` → `// DotNet Core Runtime 10.0.9`).

### Update VSO0 content hashes
After updating URLs (leave the old hashes temporarily), run a filtered build to let BuildXL compute and report the correct hashes:

```powershell

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/BuildXL](https://github.com/microsoft/BuildXL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
