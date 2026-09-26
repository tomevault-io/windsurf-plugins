---
trigger: always_on
description: Guidance for working in the C# implementation.
---

# AGENTS.md — C# / .NET 10 WinUI3 RoomRelay

Guidance for working in the C# implementation.

## What this is

Windows-only .NET 10 / WinUI 3 desktop app that:

- Captures system audio via WASAPI loopback (raw CsWin32; no NAudio).
- Resamples / converts to 48 kHz s16 stereo (pure C# pass-through when the
  device is already at 48 kHz, otherwise the Windows Media Foundation
  resampler MFT).
- Encodes to AAC-LC @ 256 kbps using the Windows Media Foundation AAC encoder
  MFT, configured with `MF_MT_AAC_PAYLOAD_TYPE = 1` so output is already
  ADTS-framed.
- Serves the ADTS stream on `http://<host>:8000/stream.aac` from a raw
  `TcpListener` HTTP/1.0 server.
- Discovers Sonos speakers via SSDP (concurrent per-NIC, IPv4 + IPv6), pulls
  user-set zone names from `ZoneGroupTopology`, filters to coordinators, and
  pushes the URL via UPnP SOAP `SetAVTransportURI` + `Play`.
- Mutes the local render endpoint while streaming so the room doesn't hear
  the PC audio twice; loopback captures pre-mute, so Sonos still gets data.

There is **no FFmpeg dependency** and **no NAudio dependency**. Everything
that touches audio goes through Windows Media Foundation or raw WASAPI via
CsWin32-generated bindings.

## Build & run

```powershell
dotnet build                                  # debug build (all 3 projects)
dotnet build -c Release                       # release build
dotnet run --project src/SonosStreaming.App   # launch the app
dotnet test                                   # run unit + integration tests
```

**Prerequisites:**
- .NET 10 SDK.
- Windows App Runtime 2.0 framework package (run
  `Get-AppxPackage *WindowsAppRuntime.2.0*` to check).

The project uses **framework-dependent** deployment (`WindowsAppSDKSelfContained=false`,
`WindowsPackageType=None`) with a **custom `Program.cs`** entry point (`DISABLE_XAML_GENERATED_MAIN`).
The Windows App Runtime is resolved via `Bootstrap.Initialize(0x00020000)` at
startup — this must happen before any WinUI types are touched.

The app binds `0.0.0.0:8000` and sends SSDP multicast on first scan — a firewall
prompt is expected on first run.

## Solution layout

```
csharp/
  SonosStreaming.sln
  Directory.Build.props                  # shared: net10.0-win, x64, unsafe, C# preview
  src/
    SonosStreaming.Core/                  # .NET library — audio, network, state, pipeline
      NativeMethods.txt                   # CsWin32 bindings list (Core)
      Audio/
        PcmFrameTypes.cs                 # PcmFrameF32, PcmFrameI16, PcmConvert
        IAudioSource.cs                  # IAudioSource interface, MixFormat record
        WasapiCaptureBase.cs             # Template-method base for WASAPI capture sources
        WasapiLoopbackSource.cs          # Raw CsWin32 WASAPI loopback + silence injection
        ProcessLoopbackSource.cs         # ActivateAudioInterfaceAsync VAD\Process_Loopback
        SyntheticSource.cs              # Sine / silence / noise test source
        Resampler.cs                    # Pass-through @ 48 kHz, MF resampler MFT otherwise
        MfAacEncoder.cs                 # Media Foundation AAC encoder MFT (CLSID_CMSAACEncMFT)
        AdtsFrameScanner.cs             # ADTS header parse + scan utility
        EndpointMuteGuard.cs            # IAudioEndpointVolume mute guard via CsWin32
        AudioEndpointMonitor.cs         # Polls default endpoint format; raises FormatChanged
        Dsp/
          GainStage.cs                  # Per-channel gain (SIMD)
          VolumeStage.cs                # Global volume scalar with soft clip
          VuMeter.cs                    # RMS + peak per channel
          BiquadEqualizer.cs            # 3-band RBJ peaking EQ
          ChannelDelay.cs              # Per-channel ring buffer delay
          SpectrumAnalyzer.cs           # MathNet FFT, 64-band log-scale, -90..0 dB
      Network/
        BroadcastChannel.cs             # Per-subscriber Channel<T> fan-out
        StreamServer.cs                 # Raw TcpListener HTTP/1.0 server
        SsdpDiscovery.cs                # SSDP M-SEARCH (concurrent per-socket, IPv4+IPv6)
        SonosController.cs              # SOAP SetAVTransportURI / Play / Stop
        TopologyResolver.cs             # ZoneGroupState → coordinators + zone names
        LocalIpResolver.cs              # UDP-connect trick for NIC selection (dual-stack)
        ISonosController.cs             # Interface for testability
        ISsdpDiscovery.cs               # Interface for testability
        ITopologyResolver.cs            # Interface for testability
        IStreamServer.cs                # Interface for testability
      State/
        AppCore.cs                      # Idle / Starting / Streaming / Stopping state machine
        AppSettings.cs                  # JSON settings persistence (%APPDATA%\RoomRelay\settings.json)
      Pipeline/
        PipelineRunner.cs               # Orchestrates capture → DSP → encode → stream → SOAP
        PipelineOptions.cs              # Config constants
    SonosStreaming.App/                  # WinUI 3 unpackaged desktop app
      NativeMethods.txt                  # CsWin32 bindings list (App)
      Program.cs                        # Custom entry: Bootstrap + Application.Start
      App.xaml / App.xaml.cs            # DI container, Serilog, tray setup

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [guicn555/RoomRelay](https://github.com/guicn555/RoomRelay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
