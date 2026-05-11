---
trigger: always_on
description: Short-form context for AI pair programmers. Read this before touching code.
---

# Perfect Bluetooth MIDI For Windows — Claude Code project brief

Short-form context for AI pair programmers. Read this before touching code.

## What this app is

Single-exe Windows app (Avalonia 11, .NET 10, win-x64) that bridges a Bluetooth LE
MIDI device to a Windows MIDI Services endpoint. Primary tested target is a
Roland FP-90X digital piano. Secondary target is any BLE MIDI peripheral that
follows the Apple 2015 BLE-MIDI 1.0 spec.

Two host-side backends, picked at startup by `WmsRuntime.EnsureInitialized`:

  - **Virtual** (preferred) — declares an app-owned UMP virtual device via
    the WMS App SDK (`Microsoft.Windows.Devices.Midi2.Endpoints.Virtual.
    MidiVirtualDeviceManager.CreateVirtualDevice`). No pre-flight loopback
    setup; the endpoint lives only while the app runs. **Requires** the WMS
    App SDK Runtime to be installed on the machine — separate ~219 MB
    download from <https://github.com/microsoft/MIDI/releases>.

  - **Loopback** (fallback) — legacy path used when the SDK runtime isn't
    installed (or the user explicitly opted in via
    `AppSettings.HostBackend = "Loopback"` in `%AppData%\PerfectBluetoothMidi\
    app.json`). Opens a pre-existing WMS loopback endpoint via the WinMM
    API. Works against the in-box WMS service alone — no extra runtime
    install needed, but the user has to create the loopback themselves
    (we try `midi loopback create` automatically if the WMS CLI is on PATH).

Flow:

    [ FP-90X piano ] <—BLE MIDI—> [ this bridge ] <—UMP or WinMM—> [ WMS endpoint ]
                                                                          ^
                                                                          |
                                                          DAW / Chrome Web MIDI /
                                                              MIDI-OX, etc.

## Repo layout

    PerfectBluetoothMidi.sln
    BUILD.bat                      ← dotnet publish wrapper → dist\PerfectBluetoothMidi.exe
    NuGet.config                   ← + local-feed entry pointing at nuget-packages/
    nuget-packages/                ← vendored Microsoft.Windows.Devices.Midi2.*.nupkg
                                     (the WMS App SDK is NOT on nuget.org —
                                      see GitHub releases). Update by dropping
                                      the new .nupkg in and bumping the
                                      PackageReference Version in the .csproj.
    README.md
    LICENSE
    docs/                          ← GitHub Pages static site
    PerfectBluetoothMidi\
        App.axaml / App.axaml.cs   ← Avalonia application shell
        Program.cs                 ← main() + CLI-vs-GUI branch
        CliHost.cs                 ← headless CLI (BLE-only; no host endpoint)
        MainWindow.axaml(.cs)      ← main GUI window + wiring + backend select
        LoopbackSetupDialog.*      ← shown only in Loopback mode if no loopback exists
        PianoKeyboard.cs           ← custom Avalonia control (on-screen piano)
        BleMidiClient.cs           ← BLE scan/connect/pair/TX/RX + TransmitChannel rewrite
        BleMidiParser.cs           ← BLE-MIDI 1.0 framing: decode + encode
        Bridge.cs                  ← glue: BLE ⇄ IHostMidiEndpoint
        IHostMidiEndpoint.cs       ← interface — the host side of the bridge
        WinMMHostEndpoint.cs       ← legacy backend (WMS loopback via WinMM)
        WmsVirtualHostEndpoint.cs  ← preferred backend (WMS virtual UMP device)
        WmsRuntime.cs              ← SDK init/detection + UMP ⇄ MIDI 1.0 helpers
        ChannelDetector.cs         ← N-ascending-notes-per-channel auto-detector
        DeviceSettings.cs          ← per-MAC settings persistence (devices.json)
        AppSettings.cs             ← global settings: theme, HostBackend, VirtualPortName
        WinMMMidi.cs               ← tiny P/Invoke wrapper over the legacy MM API
        Diag.cs                    ← verbose-logging toggle + hex helpers
        app.ico / app.manifest
    dist\                          ← build output (gitignored)

## Build

From any terminal at the repo root:

    .\BUILD.bat

or directly:

    dotnet publish PerfectBluetoothMidi\PerfectBluetoothMidi.csproj -c Release -r win-x64 ^
        --self-contained false -o dist

The exe is `dist\PerfectBluetoothMidi.exe`. No args = GUI; any recognised CLI flag
(`--scan`, `--connect`, `--detect-channels`, `--help`) = headless mode.

## Things that matter

- **Avalonia XML quirks**: `<!-- x -->` must not contain `--` inside the body.
  Attached properties must be fully qualified in XAML (`Grid.Row=` not `Row=`).
  The SDK auto-globs `*.axaml` as AvaloniaResource — do NOT add an explicit
  `<AvaloniaResource Include="**/*.axaml" />`, it causes AVLN2002 duplicate
  `x:Class`.
- **Write-mode choice**: `ResolveWriteOption` prefers `WriteWithResponse`
  because several BLE-MIDI devices (FP-90X observed) silently drop
  `WriteWithoutResponse` packets. Don't "fix" this back.
- **Proactive pairing** is done in `ConnectAsync` BEFORE enabling notifications.
  Several devices ignore MIDI on an unencrypted link while still returning
  Success at ATT — so pairing has to happen before we decide things are working.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mayerwin/Perfect-Bluetooth-MIDI-For-Windows](https://github.com/mayerwin/Perfect-Bluetooth-MIDI-For-Windows) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
