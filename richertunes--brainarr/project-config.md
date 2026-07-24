---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Brainarr is a **production-ready** multi-provider AI-powered import list plugin for Lidarr that generates intelligent music recommendations. The project supports 14 different AI providers ranging from privacy-focused local models to powerful cloud services (including 3 subscription providers and 1 CLI provider that reuse existing CLI credentials).

## Runtime & Docker Image Requirements (CRITICAL)

**Target framework**: `net8.0` — all plugins MUST target .NET 8.

**Lidarr Docker image**: Use ONLY a `.NET 8` plugins-branch image. The current pinned tag is:

```text
LIDARR_DOCKER_VERSION=pr-plugins-3.1.2.4913
```

- Image: `ghcr.io/hotio/lidarr:pr-plugins-3.1.2.4913`
- Digest: `sha256:ae0b3b14769fdfeb73fe5d9e61ebcda04edf202244bcbd6323d2fe1381154f57`
- Pinned in: `.github/lidarr_digest.txt` and `scripts/extract-lidarr-assemblies.sh`

**NEVER use `pr-plugins-2.x` tags** — those are .NET 6 images. Loading a .NET 8 plugin into a .NET 6 host causes `System.Runtime` assembly load failures and Lidarr crash-loops. The guardrail in `extract-lidarr-assemblies.sh` will catch this (fails if `System.Runtime.dll` major != 8).

When bumping the Docker image tag, update ALL of these locations:
- `.github/lidarr_digest.txt`
- `scripts/extract-lidarr-assemblies.sh` (default fallback)
- `scripts/snapshots/run-local.sh` and `run-local.ps1`
- `test-local-ci.sh`
- `scripts/verify-local.ps1` (`LidarrDockerVersion` default)

## Plugin DLL Naming Contract (CRITICAL)

**The main plugin DLL filename MUST match the glob `Lidarr.Plugin.*.dll`.** Lidarr's PluginLoader (`NzbDrone.Common/Extensions/PathExtensions.cs:334`) scans `/config/plugins/{owner}/{name}/` with `Directory.GetFiles(folder, "Lidarr.Plugin.*.dll")` — any other filename is silently ignored. No error, no warning, no log line; the plugin just never appears in `/api/v1/system/plugins`.

For Brainarr this is satisfied by `<AssemblyName>Lidarr.Plugin.Brainarr</AssemblyName>` in `Brainarr.Plugin/Brainarr.Plugin.csproj`. Don't drop that line "to clean up" — it's load-bearing.

## Plugin Packaging Policy (CRITICAL)

**The plugin package MUST contain:**
- `Lidarr.Plugin.Brainarr.dll` - Main plugin (Common **and** Abstractions are ILRepack-MERGED +
  internalized into this DLL — the "0 external `Lidarr.Plugin.*` refs" rule + the ecosystem
  standard post-ALC-fix)
- `plugin.json`
- `manifest.json`

`Lidarr.Plugin.Abstractions.dll` and `Lidarr.Plugin.Common.dll` MUST NOT ship as **sidecars** — they
are merged into the main DLL; a stray sidecar reintroduces the COR_E_INVALIDOPERATION type-identity /
ALC conflict (see `docs/dev-guide/ALC_MULTIPLUGIN_FIX.md` in Common). They are listed `[FORBIDDEN]` in
`packaging/expected-contents.txt`.

**The plugin package MUST NOT contain host-provided contract assemblies (type-identity conflicts):**
- `FluentValidation.dll`
- `Microsoft.Extensions.DependencyInjection.Abstractions.dll`
- `Microsoft.Extensions.Logging.Abstractions.dll`
- `NLog.dll`
- `System.Text.Json.dll`
- `Lidarr.*.dll` (non-plugin host assemblies)
- `NzbDrone.*.dll`

**The plugin must reference host versions of contract packages.** Use the host-version coupling tests to keep NuGet versions aligned with the Lidarr host.

**Do not rely on copying host-provided contract assemblies into the plugin output:**

```xml
<!-- OK for compile-time (runtime resolves from host); packaging MUST exclude the DLL -->
<PackageReference Include="FluentValidation" />
```

**Validation:** Run `./build.ps1 -Package` and verify the zip contains the required DLLs.

## Plugin Registration (CRITICAL — controls Lidarr System→Plugins UI visibility)

Lidarr has **two** distinct `IPlugin` interfaces, and conflating them silently breaks the System→Plugins UI:

| Interface | From | Used by |
|---|---|---|
| `NzbDrone.Core.Plugins.IPlugin` | `Lidarr.Core.dll` (host) | `/api/v1/system/plugins` — UI listing, update checks, uninstall |
| `Lidarr.Plugin.Abstractions.IPlugin` | Common (internalized via ILRepack) | TestKit `PluginSandbox` — never read by the live host |

`BrainarrPluginHost : StreamingPlugin<TModule,TSettings>` satisfies Common's contract for the bridge. It does **not** satisfy the host's `IPlugin`, so without an additional class the plugin loads fine and exposes `/api/v1/importlist/schema` but doesn't appear in System→Plugins (and can't be auto-updated/uninstalled through the UI).

`Brainarr.Plugin/Hosting/BrainarrInstalledPlugin.cs` extends the host's `NzbDrone.Core.Plugins.Plugin` to close the gap:

```csharp
public sealed class BrainarrInstalledPlugin : NzbDrone.Core.Plugins.Plugin
{
    public override string Name => "Brainarr";
    public override string Owner => "RicherTunes";
    public override string GithubUrl => "https://github.com/RicherTunes/Brainarr";
}
```

DryIoc's `RegisterMany` (in `NzbDrone.Common.Composition.Extensions.AutoAddServices`) auto-discovers this class from the loaded plugin assembly. `InstalledVersion` is derived from `AssemblyInformationalVersionAttribute` via the base class — do **not** hardcode it.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RicherTunes/Brainarr](https://github.com/RicherTunes/Brainarr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
