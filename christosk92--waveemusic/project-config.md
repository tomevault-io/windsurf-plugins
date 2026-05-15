---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

﻿# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository at a glance

WaveeMusic is a clean-room Spotify desktop client for Windows: a .NET 10 reimplementation of Spotify's Access Point, Mercury, Connect (Dealer WebSocket), SpClient (protobuf/HTTPS), and Pathfinder (GraphQL) protocols, wrapped in a WinUI 3 app. Requires a Spotify Premium account.

Top-level project READMEs are the canonical source of architectural detail — they're already detailed and current. Read them before invasive changes:

- `README.md` — features, project layout, gabo events surface, what's excluded from public source.
- `src/Wavee/README.md` — core protocol library entry points (`Session`, `DealerClient`, `PlaybackOrchestrator`, `SpClient`, `PathfinderClient`, `AudioKeyManager`).
- `src/Wavee.UI/README.md` — framework-neutral UI service layer.
- `src/Wavee.UI.WinUI/README.md` — desktop app composition, services, custom MSBuild targets, on-device AI plumbing.
- `src/Wavee.AudioHost/README.md` — out-of-process audio runtime (x64-only).
- `src/Wavee.Playback.Contracts/README.md` — IPC wire format.
- `src/Wavee.Console/README.md` — AOT CLI client (Linux/Docker friendly).
- `src/Wavee.Controls.Lyrics/README.md` — lyrics rendering control library.
- `src/Wavee/Connect/DEALER_PROTOCOL.md` and `DEALER_IMPLEMENTATION_GUIDE.md` — wire-level Dealer reference.
- `src/Wavee/OAuth/OAUTH_FLOWS.md` — Spotify OAuth analysis.

## Common commands

All commands run from the repo root.

```bash
# Build
dotnet build                       # debug
dotnet build -c Release

# Run desktop client
dotnet run --project src/Wavee.UI.WinUI

# Run console client
dotnet run --project src/Wavee.Console

# Run AudioHost manually for diagnostics (refuses to start without --standalone-dev)
dotnet run --project src/Wavee.AudioHost -p Platform=x64 -- --standalone-dev --pipe MyPipe --verbose

# Tests
dotnet test                                                              # whole solution
dotnet test test/Wavee.Tests/Wavee.Tests.csproj                               # core suite
dotnet test test/Wavee.UI.Tests/Wavee.UI.Tests.csproj                         # UI service layer
dotnet test test/Wavee.Tests/Wavee.Tests.csproj --filter "FullyQualifiedName~ShannonCipher"   # single class
dotnet test test/Wavee.Tests/Wavee.Tests.csproj --filter "FullyQualifiedName~Wavee.Tests.Connect"  # namespace

# PlayPlay decryption tests (x64-only, plain Exe harness — exits non-zero on failure, not xUnit)
dotnet run --project test/Wavee.PlayPlay.Tests -p Platform=x64

# Native publish for Wavee.Console
dotnet publish src/Wavee.Console -c Release -r win-x64
dotnet publish src/Wavee.Console -c Release -r linux-x64
dotnet publish src/Wavee.Console -c Release -r linux-arm64
```

Prerequisites: .NET 10 SDK, Windows 11 24H2 (build 26100) or later for the WinUI app, Spotify Premium account.

## High-level architecture

### Process model

The desktop app runs as **two processes** that talk over a named pipe:

```
Wavee.UI.WinUI  (any arch)  ◄── named pipe ──►  Wavee.AudioHost  (x64-only)
```

`Wavee.UI.WinUI`'s `BuildAudioHost` MSBuild target spawns an **isolated** `dotnet build` subprocess for `Wavee.AudioHost` on every WinUI build (with `Platform=x64` forced). This is deliberate, not a quirk: an in-process MSBuild reference would inherit the parent build's project-evaluation cache and could land an ARM64 NVorbis.dll next to AudioHost's x64 exe. The fresh subprocess prevents that whole class of bug. Cost: a few seconds per launch.

AudioHost is x64-only because it `LoadLibrary`s `Spotify.dll` (x86_64-native) for PlayPlay key derivation. On ARM64 Windows it runs under built-in x64 emulation. The WinUI process can be any arch.

### Why a separate audio process

1. Audio engine crashes don't take the UI down; UI can respawn the host.
2. Audio doesn't pay for the WinUI process's reflection-heavy MVVM heap; UI doesn't pay for native audio buffers.
3. Per-process arch isolation (see above).

### IPC contract

Length-prefixed JSON over named pipe: `[4 bytes big-endian length][UTF-8 JSON payload]`. Each `IpcMessage` carries `type` (kebab-case discriminator), `id` (request correlation), and `payload` (free-form `JsonElement`).

`Wavee.Playback.Contracts` is consumed two different ways on purpose:
- **Project reference** by `Wavee` and (transitively) `Wavee.UI.WinUI`.
- **Source-included** by `Wavee.AudioHost` (`<Compile Include="..\Wavee.Playback.Contracts\IpcMessages.cs">`). AudioHost has **zero project references on Wavee\* assemblies** so a stale `Wavee.Playback.Contracts.dll` can never land alongside `Wavee.AudioHost.exe` and break startup. JSON wire format means type identity across assemblies doesn't matter.

When adding a new IPC command/event: edit `src/Wavee.Playback.Contracts/IpcMessages.cs` only. Both sides pick it up automatically.

### Project layering

```
Wavee.UI.WinUI  ──►  Wavee.UI            (framework-neutral, plain C# — testable without WinUI)
                ──►  Wavee                (core protocol library, no UI)
                ──►  Wavee.Playback.Contracts
                ──►  Wavee.Controls.Lyrics
                ──►  Lyricify.Lyrics.Helper (vendored)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [christosk92/WaveeMusic](https://github.com/christosk92/WaveeMusic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
