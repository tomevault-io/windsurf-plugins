---
trigger: always_on
description: This repository contains **IoT device bindings** (drivers) for [.NET nanoFramework](https://www.nanoframework.net/) — a free, open-source platform that enables running C# on resource-constrained embedded devices (MCUs) such as ESP32 and STM32. Each binding is a standalone NuGet package targeting the `netnano1.0` framework, **not** standard .NET or .NET Core.
---

# Copilot Instructions for nanoFramework.IoT.Device

## Repository Overview

This repository contains **IoT device bindings** (drivers) for [.NET nanoFramework](https://www.nanoframework.net/) — a free, open-source platform that enables running C# on resource-constrained embedded devices (MCUs) such as ESP32 and STM32. Each binding is a standalone NuGet package targeting the `netnano1.0` framework, **not** standard .NET or .NET Core.

The bindings are adapted from the [.NET IoT repository](https://github.com/dotnet/iot) with changes needed for the nanoFramework runtime.

---

## Directory Structure

```
devices/              ← One sub-folder per device binding (primary working area)
  <DeviceName>/
    <DeviceName>.nfproj        ← Main library project (NOT .csproj)
    <DeviceName>.sln           ← Solution file (includes main, samples, tests)
    <DeviceName>.nuspec        ← NuGet packaging descriptor
    packages.config            ← NuGet package references
    packages.lock.json         ← Locked NuGet packages
    version.json               ← Nerdbank.GitVersioning configuration
    category.txt               ← Device category tags (for README auto-generation)
    Settings.StyleCop          ← StyleCop rules
    README.md                  ← Device documentation
    Properties/AssemblyInfo.cs ← Assembly metadata
    samples/                   ← Sample application project (.nfproj)
    tests/                     ← Unit test project (.nfproj, nano.runsettings)

src/                  ← Tooling and helper projects (not device bindings)
  nanoFramework.IoT.Device.CodeConverter/  ← Converts .NET IoT code to nanoFramework
  device-listing/              ← Generates the devices/README.md
  devices_generated/           ← Auto-generated bindings (do not manually edit)

.pipeline-assets/
  pipeline-build-solutions.PS1 ← Azure Pipelines build script (Windows/MSBuild)

azure-pipelines.yml   ← CI/CD configuration (Azure Pipelines, Windows)
```

---

## nanoFramework vs Standard .NET — Critical Differences

**These constraints apply to ALL code in `devices/` (library and sample code):**

### What is NOT available in nanoFramework
- **No `Console`** — use `Debug.WriteLine()` instead (from `System.Diagnostics`)
- **No generic collections** — no `List<T>`, `Queue<T>`, `Dictionary<K,V>`. Use `ArrayList` with casts
- **No multidimensional arrays `[,]`** — use jagged arrays `[][]` instead
- **No `Enum.GetValues()` / `Enum.IsDefined()`** — use switch statements or remove the check
- **No `Span<byte>`** — use `SpanByte` (nanoFramework type)
- **Limited LINQ** — avoid LINQ in library code
- **No `stackalloc` in many contexts** — allocate with `new byte[]` instead
- **No generics in all scenarios** — generics are partially supported; test carefully
- **No `Thread.Sleep(Timeout.Infinite)` in samples** — use a `while(true)` loop

### nanoFramework-specific APIs
- `SpanByte` replaces `Span<byte>` for I2C/SPI buffers
- `System.Diagnostics.Debug.WriteLine()` for output (shows in VS debug window / serial output)
- `nanoFramework.Hardware.Esp32.Configuration.SetPinFunction()` to configure GPIO pins on ESP32
- `nanoFramework.TestFramework` for unit tests (attributes: `[TestClass]`, `[TestMethod]`, `[Setup]`)
- `UnitsNet` NuGet packages (nanoFramework-specific builds, e.g. `nanoFramework.UnitsNet.Temperature`) for physical measurements

### Vector3 / System.Numerics
The `devices/System.Numerics/` implementation uses `double` components (X, Y, Z) — **not** `float` as in standard .NET.

---

## Project Conventions

### Naming
- **NuGet package ID**: `nanoFramework.Iot.Device.<DeviceName>`
- **Assembly name**: `Iot.Device.<DeviceName>`
- **Root namespace**: `Iot.Device.<DeviceName>`
- **Project GUID type**: `{11A8DD76-328B-46DF-9F39-F559912D0360}` (nanoFramework project type)

### File Headers
Every `.cs` source file must start with:
```csharp
// Licensed to the .NET Foundation under one or more agreements.
// The .NET Foundation licenses this file to you under the MIT license.
```

### Strong Name Signing
All library projects are signed with the shared `devices/key.snk` key. Each `.nfproj` must have:
```xml
<SignAssembly>true</SignAssembly>
<AssemblyOriginatorKeyFile>..\key.snk</AssemblyOriginatorKeyFile>
<DelaySign>false</DelaySign>
```

### StyleCop
StyleCop is enforced via `StyleCop.MSBuild` NuGet. Errors (not just warnings) are expected to be fixed. The `Settings.StyleCop` file in each device folder configures the rules.

### Code Style
- XML documentation comments (`/// <summary>`) are required on all public APIs
- Constructor pattern: accept `I2cDevice` or `SpiDevice` as the primary argument
- Use `IDisposable` and dispose the underlying device in `Dispose()`
- Constants for default I2C addresses: `public const byte DefaultI2cAddress = 0x...;`

---

## I2C Sample Code Pattern

For ESP32, always configure I2C GPIO pins before creating `I2cDevice`. The standard pins for I2C bus 1 are GPIO21 (SDA) and GPIO22 (SCL):

```csharp

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nanoframework/nanoFramework.IoT.Device](https://github.com/nanoframework/nanoFramework.IoT.Device) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
