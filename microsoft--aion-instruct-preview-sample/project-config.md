---
trigger: always_on
description: You are working in the **consumer sample** for the **Aion Instruct Preview SDK**: an on-device
---

# Copilot instructions — Aion Instruct Preview Chat sample

You are working in the **consumer sample** for the **Aion Instruct Preview SDK**: an on-device
language model that runs locally on the Copilot+ PC NPU — QNN on Snapdragon (ARM64). x64 (Intel/AMD,
OpenVINO) support is coming soon. A certified NPU EP is required (no CPU fallback, no cloud calls).
This repo shows a
third-party developer how to (1) consume the SDK's framework package, (2) wire it into their own
app, and (3) build + run a working chat app. Use this file to orient fast; the
[`README.md`](../README.md) has the deep dives and is the source of truth — cross-references below
point at its sections.

## What this repo contains

Three consumption patterns of the **same** SDK, so a developer can copy whichever matches their app:

| Path | Project | Identity model | Framework dependency taken |
| --- | --- | --- | --- |
| **Packaged WinUI 3** (canonical) | `AionInstructPreview.Chat.csproj` (repo root) | MSIX package identity | **Build-time**: SDK targets inject a `<PackageDependency>` into `Package.appxmanifest` |
| **Unpackaged console** | `unpackaged-console/AionInstructPreview.Chat.Console.csproj` | none (plain exe) | **Runtime**: `FrameworkDependency.cs` via `TryCreatePackageDependency` / `AddPackageDependency` |
| **Unpackaged WPF** | `unpackaged-wpf/AionInstructPreview.Chat.Wpf.csproj` | none (plain exe) | **Runtime**: same dynamic-dependency pattern as console |

The root WinUI app is the one most consumers should copy. The unpackaged variants exist for apps
that can't adopt MSIX identity; they take the same framework dependency at runtime instead of via a
manifest.

## The mental model (read this before changing build wiring)

There are **two distinct things** with similar names; never conflate them:

1. **The SDK NuGet — `AionInstructPreview.Text.Framework`** (a `PackageReference`). This is
   **build-time only**. Its build hooks:
   - `AionInstructPreview.Text.Framework.props` adds `AionInstructPreview.Text.winmd` to
     `$(CsWinRTInputs)` so **CsWinRT** projects the WinRT runtimeclasses into C# types you can call.
   - `AionInstructPreview.Text.Framework.targets` injects the runtime `<PackageDependency>` into a
     packaged app's `AppxManifest` (no-ops for unpackaged apps, which have no manifest).
2. **The framework MSIX — `Microsoft.AionInstructPreview.Framework.1.0`** (installed system-wide).
   This is the **runtime** that actually holds `AionInstructPreview.Text.dll`, `muffinapi.dll`, and
   the ONNX model files. It is **not on any public feed** — it ships only as a **GitHub release
   asset** of this repo (by design; this is a developer preview). `Bootstrap.ps1` downloads and
   installs it.

A build error almost never means a *runtime* component is missing, and vice-versa. See
README → [Prerequisites](../README.md#prerequisites).

## Build & run the sample

**Prereqs:** .NET 9 SDK + **Developer Mode** on. That's it — **no Visual Studio and no
registry-installed Windows SDK are required**; everything else (Windows App SDK, CsWinRT, SDK build
tools, the Windows metadata ref pack, MSIX loose-layout tooling) is restored from NuGet.

**One-shot quickstart** (downloads + installs the framework MSIX from this repo's GitHub release via
`gh`, drops the SDK NuGet into `./nuget-local/`, then builds and launches):

```powershell
./Bootstrap.ps1
```

**Manual build + run** (packaged WinUI app). This preview targets **ARM64 Snapdragon Copilot+ PCs
(NPU via QNN)** — build with `-p:Platform=ARM64`. x64 (Intel/AMD) support is coming soon. The csproj
defaults `$(Platform)` to the box's *native* arch, so a bare `dotnet build` on an ARM64 box already
does the right thing.

```powershell
dotnet run --project AionInstructPreview.Chat.csproj --launch-profile "AionInstructPreview.Chat" -c Release -p:Platform=ARM64   # Snapdragon (QNN NPU)
```

`dotnet run` registers the build output as a **loose-layout development package** (this is what
gives the app the package identity it needs for cross-package WinRT activation) and launches it —
no signing cert, no separate `Add-AppxPackage`. Developer Mode is required for that registration.

**Unpackaged variants:** `cd unpackaged-console` (or `unpackaged-wpf`) then `./Run.ps1` — it checks
the framework package is installed, builds, and runs. These have no MSIX identity and pull the
framework dependency at runtime.

**Visual Studio:** the F5 path is supported on **Visual Studio 2026 (18.x) only**, with the
**(MSIX)** launch profile (never the `Project` profile — that runs the bare exe without identity and
crashes with `REGDB_E_CLASSNOTREG`). VS 2022 (17.x) cannot F5 this Windows App SDK 2.0 app. Select the
**Solution Platform set to ARM64** — a non-ARM64 platform makes F5 report
*"The project needs to be deployed. Please enable Deploy in the Configuration Manager"* even though
Deploy is enabled. Exact required VS 2026 components are in
README → [Visual Studio 2026 setup](../README.md#visual-studio-2026-setup).

## Consume the SDK in your OWN app

Minimum wiring to call the model from a new project (full guide:
README → [Use Aion Instruct Preview in your own app](../README.md#use-aion-instruct-preview-in-your-own-app)):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/Aion-Instruct-Preview-Sample](https://github.com/microsoft/Aion-Instruct-Preview-Sample) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
