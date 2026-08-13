---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

Resonalyze is a Windows desktop application (WinForms, .NET 10) for acoustic measurements: impulse/frequency response, loopback-referenced time alignment, live transfer functions, EQ design, and virtual DSP crossover simulation. The SDK version is pinned in `global.json`.

## Commands

```powershell
dotnet restore source/Resonalyze.sln
dotnet build source/Resonalyze.sln --configuration Release
dotnet run --project source/Resonalyze.csproj

# All tests
dotnet test source/Resonalyze.sln -c Release

# One test project
dotnet test tests/Resonalyze.Dsp.Tests/Resonalyze.Dsp.Tests.csproj

# One test class or method
dotnet test tests/Resonalyze.Dsp.Tests/Resonalyze.Dsp.Tests.csproj --filter "FullyQualifiedName~TransferFunctionTests"

# Performance profiling build (defines TRACY_ENABLE, references Tracy-CSharp)
dotnet run --project source/Resonalyze.csproj -c Tracy
```

Platform constraint: `source/` (the app), `audio/Resonalyze.Audio`, `tests/Resonalyze.Audio.Tests/` and `tests/Resonalyze.App.Tests/` target `net10.0-windows` and only build/run on Windows (WASAPI/ASIO/MME are Windows-only). `dsp/` and `tests/Resonalyze.Dsp.Tests/` target plain `net10.0` and are cross-platform — on a Linux environment, only the DSP library and its tests can be built and run.

## Architecture

Three projects, with deliberate boundaries (`Dsp` ⟂ `Audio`; the app depends on both):

- **`dsp/Resonalyze.Dsp`** — pure, UI-free signal-processing library. Depends only on MathNet.Numerics and YamlDotNet. Contains FFT/spectrum analysis, windowing, transfer functions, minimum phase, excess delay, time-alignment analysis, biquad/crossover filters, the EQ auto-tuner, and PEQ profile import/export formats (Equalizer APO, REW, MiniDSP, CamillaDSP, EasyEffects, generic CSV — all implementing `IEqProfileFormat`). Every `PeqBand.Q` in the library is RBJ-cookbook Q, which is what `PeakingBiquad` realizes and what the fitting, previews and profile formats all assume; `PeqQConventions` restates a band for a device that reads Q as Symmetric (Zölzer/DAFX) or Classic, and is applied only where numbers leave for such a device (the tuning sheets), never to the internal representation. The conventions are one filter family differing by a Q scale of `10^(±gain/40)`, so conversion is exact — `PeqQConventionTests` pins it against an independently written Zölzer section and against REW's published half-gain bandwidths. The app hands measurement data to this layer through `IImpulseMeasurement` (impulse response + peak index + sample rate).
- **`audio/Resonalyze.Audio`** — owns all audio drivers/devices (WASAPI Shared/Exclusive, ASIO, MME), format negotiation, capture/playback lifecycle, PCM decoding, diagnostics and warm-up. NAudio is confined here (declared `PrivateAssets="compile"` so `using NAudio` does not compile in the app). Low-level device types are `internal`; the measurement layer talks only to the neutral abstraction: `IAudioSessionFactory` + `IAudioDuplexSession`/`IAudioStreamingSession`/`IAudioPlaybackSession` and backend-neutral DTOs (`AudioSessionRequest`, `AudioPlaybackSignal`, `AudioCaptureResult`, `AudioSessionDiagnostics`, `AudioEndpointDescriptor`, `AudioFormat`, `PlaybackChannel`). Backends are chosen by the persisted `AudioBackend` enum inside `AudioBackendRegistry` (the only backend dispatch — no `switch (AudioBackend)` in `source/`).
- **`source/Resonalyze`** — the WinForms app: composition root, measurement lifecycle, and plotting.

Inside `source/`, the flow is: signal generation (`Measurements/` — `ExponentialSineSweep`, `NoiseSignal` produce float data only) → an audio session opened via `IAudioSessionFactory` (composition root in `Shell/Form1` builds `AudioBackendRegistry.CreateDefault()` and injects the factory into `ExpSweepMeasurement`, `NoiseMeasurement`, the signal generator and warm-up) → analysis via the Dsp library → plot presentation (`Plotting/` — `PlotModelFactory` builds OxyPlot models, `OxyPlotAdapter` hosts them). Microphone and loopback are always channels of ONE input device, so timing stays sample-synchronous.

Key structural points:

- **`Shell/Form1` is the hub**, split into partial classes by concern (`Form1.Measurement.cs`, `Form1.Plotting.cs`, `Form1.History.cs`, `Form1.Compare.cs`, etc.). The `Mode` enum in `Form1.cs` defines all analysis modes (frequency/phase/group delay/waterfall/burst decay/live spectrum/time alignment/EQ wizard/signal generator/virtual crossover); `ModeSwitching/ModeController` orchestrates tab switches.
- **`Options/`** holds one settings panel per mode (`FROptions`, `IROpt`, `GDOpt`, ...), docked into the shell via `Shell/DockedModeSettingsHost`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DIMOSUS/Resonalyze](https://github.com/DIMOSUS/Resonalyze) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
