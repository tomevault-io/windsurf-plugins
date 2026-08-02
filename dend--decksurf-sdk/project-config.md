---
trigger: always_on
description: DeckSurf SDK is an open-source .NET library for managing Elgato Stream Deck devices. It provides device enumeration, button image management, screen control, and a plugin architecture for third-party developers.
---

# CLAUDE.md — DeckSurf SDK

## Project Overview

DeckSurf SDK is an open-source .NET library for managing Elgato Stream Deck devices. It provides device enumeration, button image management, screen control, and a plugin architecture for third-party developers.

- **Target framework**: .NET 10
- **Cross-platform**: Core functionality works on Windows, macOS, and Linux. Windows-only APIs are guarded with `[SupportedOSPlatform("windows")]`.
- **Author**: Den Delimarsky

## Repository Structure

```
src/
  DeckSurf.SDK/              # Main SDK library
    Core/                    # DeviceManager, DeviceRegistry
    Exceptions/              # Custom exception hierarchy
    Interfaces/              # IConnectedDevice, IDeckSurfCommand, IDeckSurfPlugin
    Models/                  # Domain types, device implementations
      Devices/               # Per-device implementations + base classes
    Util/                    # ImageHelper, DataHelper, ConfigurationHelper
  DeckSurf.SDK.Tests/        # xUnit test project
  DeckSurf.SDK.StartBoard/   # Example/demo application
```

## Build and Test

```bash
cd src
dotnet build DeckSurf.SDK.sln        # Build all projects
dotnet test DeckSurf.SDK.Tests/      # Run tests with coverage enforcement
```

- Build uses `TreatWarningsAsErrors` — all warnings must be resolved.
- Tests enforce a minimum **50% line coverage** threshold via coverlet. The build will fail if coverage drops below this.
- Windows-only code (marked with `[SupportedOSPlatform]`) is excluded from coverage metrics.

## Code Style and Linting

- **StyleCop.Analyzers** enforces code style (ordering, documentation, naming).
- **Microsoft.CodeAnalysis.NetAnalyzers** enforces code quality (CA rules).
- `.editorconfig` at `src/` root defines formatting, naming conventions, and analyzer severity.
- `TreatWarningsAsErrors` is enabled — all warnings are build errors.
- All public and interface members must have XML documentation (`documentExposedElements` and `documentInterfaces` are enabled in `stylecop.json`).
- Use `this.` prefix for instance member access (project convention).
- Copyright header required on all `.cs` files (SA1633 enforced as error).
- Elements must be separated by blank lines (SA1516).
- Members must be ordered by kind (fields, constructors, events, properties, methods) and access level (public before private) per SA1201/SA1202.

### Analyzer Suppression Policy

- **Suppressing a warning is never the default.** Fix the code instead.
- A suppression requires **strong justification** — document the reason in a code comment or the `.csproj` NoWarn line.
- Legitimate suppression reasons: modern C# syntax not yet recognized by analyzers (e.g., SA1010 for collection expressions), deliberate architectural grouping (e.g., SA1649 for NativeStructures.cs), or rules that conflict with project conventions (e.g., SA1101 — this project uses `this.` prefix).
- Before adding a global suppression to NoWarn, try a targeted `#pragma warning disable` with a justification comment.
- When reviewing PRs, challenge any new suppression — the bar is "would this confuse a future contributor?"

## Synchronous API Design

All device I/O methods are **synchronous only**. This is a deliberate design decision:

- **HidSharp 2.1.0 has no true async I/O primitives.** USB HID writes are inherently blocking at the hardware level — there is no `WriteAsync` that avoids thread blocking.
- **Libraries must not lie about being async.** Wrapping synchronous calls in `Task.Run` is an anti-pattern for library code (per Stephen Toub / Microsoft guidance). It creates false async semantics, causes thread pool starvation under load, and provides fake `CancellationToken` support that can't actually cancel a USB write mid-transfer.
- **The application should own the threading decision.** Consumers who need to avoid blocking the UI thread can use `Task.Run` at the call site — they understand their threading model, the SDK does not.

If a future version of HidSharp (or a replacement library) provides true async I/O (e.g., `Stream.WriteAsync` backed by OS-level async), the SDK should adopt real async methods at that point. Until then, sync-only is the honest approach.

**Never add `*Async` methods that internally wrap sync calls in `Task.Run`.** This was tried and reverted after expert review.

## Architecture Conventions

### Device Hierarchy

```
ConnectedDevice (abstract, implements IConnectedDevice)
├── JpegButtonsDevice (abstract) — JPEG/Rotate180 devices
│   ├── StreamDeckOriginal, StreamDeckOriginal2019, StreamDeckMK2
│   ├── StreamDeckXL, StreamDeckXL2022
├── BitmapButtonsDevice (abstract) — BMP/Rotate270 devices
│   ├── StreamDeckMini, StreamDeckMini2022
├── ScreenDevice (abstract) — Devices with LCD screens
│   ├── StreamDeckNeo, StreamDeckPlus
```

- Add new devices by subclassing the appropriate base class and registering in `DeviceRegistry`.
- Device-specific properties only (Model, ButtonCount, ButtonResolution, ButtonColumns, ButtonRows).
- Shared protocol logic lives in base classes to avoid duplication.

### Adding a New Device Model


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dend/decksurf-sdk](https://github.com/dend/decksurf-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
