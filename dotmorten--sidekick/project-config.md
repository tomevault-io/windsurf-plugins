---
trigger: always_on
description: This application is a .NET worker service that renders a dashboard onto an e-paper display and
---

# Copilot instructions for SideKick

This application is a .NET worker service that renders a dashboard onto an e-paper display and
integrates with classic Outlook on Windows. Settings are managed via a Microsoft.UI.Reactor-based
window that starts hidden and is surfaced through a tray icon.

The codebase is structured as a solution with the worker service and the display driver library.

## Build and run commands

- Restore packages: `dotnet restore SideKick.slnx`
- Build the full solution with the .NET CLI: `dotnet build SideKick.slnx -p:RuntimeIdentifier=win-arm64`
- Build the worker directly: `dotnet build Worker\Worker.csproj -p:RuntimeIdentifier=win-x64`
- Build the worker with screenshot-safe demo calendar and TODO data: `dotnet build Worker\Worker.csproj -p:RuntimeIdentifier=win-x64 -p:DefineConstants=DEMO`
- Build the full solution the same way Visual Studio does: `& "C:\Program Files\Microsoft Visual Studio\18\Enterprise\MSBuild\Current\Bin\MSBuild.exe" SideKick.slnx /t:Build "/p:Configuration=Debug;Platform=ARM64"`
- Run the worker service interactively: `dotnet run --project Worker\Worker.csproj -a arm64`

There are currently no test projects in `SideKick.slnx` and no repo-specific lint command/configuration.

## Architecture

`SideKick.slnx` contains the production projects:

- `Providers` defines the calendar and TODO provider contracts and immutable entry records used by the app and provider implementations.
- `OutlookClassic` implements both provider contracts through classic Outlook COM automation and owns Outlook discovery, synchronization, and connection state.
- `WebCalendar` implements the calendar provider contract for configurable HTTP, HTTPS, and `webcal` iCalendar feeds. It expands recurring events with Ical.Net and refreshes configured feeds every 15 minutes.
- `Worker` is a .NET worker service that renders the clock/calendar UI and drives the e-paper display. `Program.cs` wires the provider interfaces, display settings service, FT232H factory, preview store, `Worker`, and Reactor settings window with DI. The worker renders on demand from trigger scheduling: startup, whole-minute clock ticks, provider changes, calendar item start/end boundaries, display settings changes, preview activation, and display reconnect retries.
- `Worker/Demo/DemoMockProvider.cs` is compiled only when the `DEMO` symbol is defined. System settings exposes separate Calendar and ToDo provider tabs where any number of available providers can be enabled; Worker merges entries from all enabled providers.
- Providers can optionally implement `IConfigurableProvider`; System settings renders their string-list configuration fields generically and persists values by provider and field ID.
- `EPaper` contains the display abstraction and hardware drivers. `IEPaperDisplay`/`IGrayscaleEPaperDisplay` define the public display contract, `EPaperDisplays` is the public factory surface, and the display implementations use internal FT232H SPI/GPIO buses.

`ReferenceSource` contains external Waveshare/Arduino reference material used for driver behavior; do not treat it as product source unless intentionally porting from the reference implementation.

## Codebase conventions

- Use file-scoped namespaces, nullable annotations, implicit usings where enabled, primary constructors for simple services, and sealed classes/records where the current code does.
- Add new displays through `IEPaperDisplay`/`IGrayscaleEPaperDisplay` and expose them from `EPaperDisplays`. Display constructors are internal and accept bus abstractions so hardware access stays behind the factory layer.
- E-paper frames are packed buffers: `EPaperFrame` is 1 bit per pixel with MSB-first byte packing, and `EPaperGrayFrame` is 2 bits per pixel. Partial update regions must be aligned with `EPaperRectangle.AlignToByteColumns`.
- Hardware operations surface failures explicitly. Existing display implementations validate initialization, dimensions, and BUSY timeouts instead of silently continuing.
- The async display APIs wrap synchronous SPI/GPIO operations and usually return `ValueTask.CompletedTask`; still thread cancellation tokens through reset/delay/wait paths.
- `OutlookClassicProvider` uses classic Outlook COM automation from the interactive desktop worker process, not Microsoft Graph or Entra ID. It depends on Outlook being installed and running in the same user session as the worker.
- `OutlookCalendarSync` configuration currently controls whether sync is enabled, the fallback minimum lookahead window, the hourly refresh cadence, and how often the worker probes for a running Outlook process. The sync service now continues pulling future appointments until it has enough items to fill the current display layout instead of stopping strictly at 24 hours.
- Display selection and orientation are persisted under `%LOCALAPPDATA%\SideKick\display-settings.json`. Supported orientations are the full 0/90/180/270 set for hardware output, while the settings-window preview stays upright and only changes between portrait and landscape aspect ratios.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dotMorten/SideKick](https://github.com/dotMorten/SideKick) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
