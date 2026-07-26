---
trigger: always_on
description: This is the **Base Class Library (BCL)** for [.NET nanoFramework](https://www.nanoframework.net/) — a free, open-source platform that implements .NET for resource-constrained embedded devices (microcontrollers). The library produces `mscorlib.dll`, the core runtime library for nanoFramework, and is analogous to `System.Private.CoreLib` in full .NET.
---

# Copilot Instructions for nanoFramework CoreLibrary

## Repository Overview

This is the **Base Class Library (BCL)** for [.NET nanoFramework](https://www.nanoframework.net/) — a free, open-source platform that implements .NET for resource-constrained embedded devices (microcontrollers). The library produces `mscorlib.dll`, the core runtime library for nanoFramework, and is analogous to `System.Private.CoreLib` in full .NET.

The BCL is split into two NuGet packages:
- **`nanoFramework.CoreLibrary`** (`nanoFramework.CoreLibrary/` project) — includes `System.Reflection` API.
- **`nanoFramework.CoreLibrary.NoReflection`** (`nanoFramework.CoreLibrary.NoReflection/` project) — excludes `System.Reflection` to save flash space on constrained targets.

## Project System

- Projects use the **nanoFramework project system** with `.nfproj` file extension (not `.csproj`). The project type GUIDs are `{11A8DD76-328B-46DF-9F39-F559912D0360};{FAE04EC0-301F-11D3-BF4B-00C04F79EFBC}`.
- The solution file is `nanoFramework.CoreLibrary.sln`.
- The output assembly name is `mscorlib` (set via `<AssemblyName>mscorlib</AssemblyName>` and `<IsMscorlib>true</IsMscorlib>`).
- The target framework version is `v1.0` (nanoFramework's own TFM, not standard .NET).
- This is a **core assembly** (`<IsCoreAssembly>true</IsCoreAssembly>` and `<NF_IsCoreLibrary>True</NF_IsCoreLibrary>`), meaning it has no external managed dependencies.

## Key Conventions

### Native Calls
Many methods are implemented natively in the nanoFramework C++ runtime (nanoCLR). These are declared with `extern` and `[MethodImpl(MethodImplOptions.InternalCall)]`:

```csharp
using System.Runtime.CompilerServices;

[MethodImpl(MethodImplOptions.InternalCall)]
public static extern bool Equals(String a, String b);
```

When implementing new methods that require native support, mark them with `[MethodImpl(MethodImplOptions.InternalCall)]` and `extern`. The native counterpart must be implemented in the [nf-interpreter](https://github.com/nanoframework/nf-interpreter) repository.

### Reflection Conditional Compilation
Code that depends on reflection must be wrapped with `#if NANOCLR_REFLECTION`:

```csharp
#if NANOCLR_REFLECTION
[DebuggerDisplay("Count = {Count}")]
#endif // NANOCLR_REFLECTION
```

The `NANOCLR_REFLECTION` constant is defined in the `nanoFramework.CoreLibrary` project but **not** in `nanoFramework.CoreLibrary.NoReflection`.

### Namespace Layout
- Source files live under `nanoFramework.CoreLibrary/System/` and `nanoFramework.CoreLibrary.NoReflection/System/`, mirroring the `System` namespace hierarchy.
- Sub-namespaces map to subdirectories: `Collections/`, `Threading/`, `Reflection/`, `Diagnostics/`, `Globalization/`, `IO/`, `Runtime/CompilerServices/`, etc.

### XML Documentation
All public APIs must have complete XML documentation comments (`<summary>`, `<param>`, `<returns>`, `<exception>`, etc.). Keep them concise; avoid adding message strings to exceptions to preserve assembly/memory size. Example:

```csharp
// (no message to preserve assembly size/memory consumption)
throw new NotSupportedException();
```

### License Header
Every source file starts with:

```csharp
// Licensed to the .NET Foundation under one or more agreements.
// The .NET Foundation licenses this file to you under the MIT license.
```

### Pragma Suppression
Some `#pragma warning disable/restore` directives for CS0659, CS0661, and S1206 (SonarAnalyzer) are used where native implementations handle `GetHashCode`. Follow this pattern when adding types that have native hash code support.

## Building

Build uses **Azure Pipelines** (`azure-pipelines.yml`), not GitHub Actions. The only GitHub Actions workflow is `generate-changelog.yml`, which triggers on version tags.

For local builds:
- Install the **nanoFramework Visual Studio extension** which brings the nanoFramework project system and `MSBuildExtensionsPath\nanoFramework\v1.0\`.
- Open `nanoFramework.CoreLibrary.sln` in Visual Studio (Windows only).
- Restore NuGet packages before building.
- The pipeline uses `windows-latest` and references the shared [nf-tools](https://github.com/nanoframework/nf-tools) template library.
- Versioning is managed by **Nerdbank.GitVersioning** (`version.json`); the current version prefix is `1.17`.
- The assembly is strong-named using `key.snk`.

## Testing

### Framework
Tests use the [nanoFramework.TestFramework](https://github.com/nanoframework/nanoFramework.TestFramework) (included as a git submodule at `nanoFramework.TestFramework/`). Tests run on the **nanoCLR Win32 emulator** (no real hardware required by default).

### Test Layout
- All test projects live under `Tests/`, one project per test area (e.g., `Tests/NFUnitTestSystemLib/`, `Tests/NFUnitTestArray/`, etc.).
- Test class files are named `UnitTest<Feature>.cs`.
- Each test class uses `[TestClass]` and each test method uses `[TestMethod]`.
- Data-driven tests use `[DataRow(...)]`.

### Critical Setup for Test Projects

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nanoframework/CoreLibrary](https://github.com/nanoframework/CoreLibrary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
