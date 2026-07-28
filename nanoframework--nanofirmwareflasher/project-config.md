---
trigger: always_on
description: This repository contains the **.NET nanoFramework Firmware Flasher** (`nanoff`), a cross-platform .NET tool and library for flashing firmware images to nanoFramework-supported microcontroller targets. It is published both as:
---

# Copilot Instructions for nanoFirmwareFlasher

## Repository Overview

This repository contains the **.NET nanoFramework Firmware Flasher** (`nanoff`), a cross-platform .NET tool and library for flashing firmware images to nanoFramework-supported microcontroller targets. It is published both as:

- A **.NET global tool** (`dotnet tool install -g nanoff`)
- A **NuGet library** (`nanoFramework.Tools.FirmwareFlasher`) for embedding in third-party apps

Firmware images are fetched from **Cloudsmith** repositories (`nanoframework-images` for stable, `nanoframework-images-dev` for preview, `nanoframework-images-community-targets` for community targets).

---

## Solution Structure

```
nanoFirmwareFlasher.sln
├── nanoFirmwareFlasher.Library/   # Core library (NuGet: nanoFramework.Tools.FirmwareFlasher)
│   ├── Esp32Serial/               # Native C# ESP32 serial bootloader protocol implementation
│   ├── DeploymentHelpers/         # Device helper utilities
│   ├── FileDeployment/            # File-based deployment support
│   ├── NetworkDeployment/         # Network-based deployment support
│   ├── Exceptions/                # Custom exception types
│   └── ...                        # Per-platform operations, firmware packages, utilities
├── nanoFirmwareFlasher.Tool/      # CLI entry point (command: nanoff)
│   ├── Program.cs                 # Main async entry point, argument parsing
│   ├── Options.cs                 # CommandLine options (CommandLineParser library)
│   └── *Manager.cs                # Per-platform CLI managers (Esp32, Stm32, TI, Silabs)
├── nanoFirmwareFlasher.Tests/     # Unit tests (MSTest)
├── lib/                           # Bundled third-party CLI tools (jlink, stlink, silink, uniflash, esp32 bootloaders)
└── Samples/                       # Sample apps demonstrating library usage
```

---

## Supported Platforms and Connections

| Platform        | Connection Method           | Key Classes                              |
|-----------------|-----------------------------|------------------------------------------|
| ESP32 / S2 / S3 | Serial (native C# protocol) | `EspTool`, `Esp32Operations`, `Esp32Firmware` |
| STM32           | JTAG, DFU                   | `StmJtagDevice`, `StmDfuDevice`, `Stm32Operations`, `Stm32Firmware` |
| TI CC13x2/CC26x2| TI Uniflash CLI             | `CC13x26x2Operations`, `CC13x26x2Firmware` |
| Silabs Giant Gecko | J-Link / silink CLI      | `JLinkOperations`, `JLinkFirmware`, `SilinkCli` |

> **Important:** ESP32 support uses a native C# implementation of the Espressif serial bootloader protocol (`EspTool` / `Esp32Serial/`). The `esptool` Python tool is no longer used (removed as of April 2026).

---

## Key Architectural Patterns

### Output
All output goes through the `OutputWriter` static class, which uses `AsyncLocal<IOutputWriter>` to support per-test output redirection. Never write directly to `Console`; use `OutputWriter.Write()` / `OutputWriter.WriteLine()`.

### Exit Codes
All operations return an `ExitCodes` enum value. Return `ExitCodes.OK` (0) on success. Errors are in ranges:
- `E1000` – DFU errors
- `E2000` – nanoDevice errors
- `E4000` – ESP32 errors
- `E5000` – STM32/JTAG errors
- `E6000` – COM port errors
- `E7000` – TI errors
- `E8000` – J-Link errors
- `E9000` – General/application errors

### Firmware Packages
`FirmwarePackage` is the abstract base class for all firmware. Concrete subclasses: `Esp32Firmware`, `Stm32Firmware`, `JLinkFirmware`, `CC13x26x2Firmware`. Firmware is downloaded from Cloudsmith, cached under `~/.nanoFramework/fw_cache/`.  
`FirmwarePackage.LocationPathBase` is settable (for tests) via its internal setter.

### Telemetry
`NanoTelemetryClient` wraps Application Insights. The connection string is loaded from `appsettings.json` in the Tool project.

### Namespace
All classes use the namespace `nanoFramework.Tools.FirmwareFlasher`.

---

## Building

```bash
dotnet build nanoFirmwareFlasher.sln
```

- The library targets **net8.0** and **net472**.
- The tool targets **net8.0**.
- Tests target **net8.0**.
- NuGet packages use locked restore (`packages.lock.json`). Lock mode is enforced in CI (`TF_BUILD` or `ContinuousIntegrationBuild` env vars). When adding/updating NuGet dependencies locally, run `dotnet restore` to update the lock file.

---

## Testing

```bash
dotnet test nanoFirmwareFlasher.Tests/nanoFirmwareFlasher.Tests.csproj
```

- Uses **MSTest** (`MSTest.TestFramework`, `MSTest.TestAdapter`).
- Tests redirect output using `OutputWriter.SetOutputWriter()` (internal API).
- `FirmwarePackage.LocationPathBase` can be set per test to isolate firmware cache.
- Parallel test execution is supported because of `AsyncLocal` usage in `OutputWriter` and `FirmwarePackage`.
- Most tests that exercise hardware or network access are integration tests and may be skipped in CI environments without physical devices.

---

## Coding Conventions (from `.editorconfig`)

- **File encoding**: `utf-8-bom`, CRLF line endings.
- **License header** on every `.cs` file:
  ```csharp
  // Licensed to the .NET Foundation under one or more agreements.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nanoframework/nanoFirmwareFlasher](https://github.com/nanoframework/nanoFirmwareFlasher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
