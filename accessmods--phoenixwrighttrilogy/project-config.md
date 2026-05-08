---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a screen reader accessibility mod for Phoenix Wright: Ace Attorney Trilogy using MelonLoader. The mod outputs game text (dialogue, menus, UI elements) directly to screen readers via the UniversalSpeech library, with SAPI fallback for users without a screen reader.

## Build Commands

```bash
# Build the mod (output goes to game's Mods folder automatically)
cd AccessibilityMod
dotnet build -c Release

# The post-build target copies AccessibilityMod.dll to $(GamePath)\Mods
# and copies Data/* files to $(GamePath)\UserData\AccessibilityMod\
```

## Utility Projects

```bash
# Validate translations against English strings.json
cd LocalizationValidator
dotnet run -- ja                    # Validate Japanese
dotnet run -- fr --strict           # Validate French, treat warnings as errors

# Build the standalone installer (publishes self-contained .exe)
cd Installer
dotnet publish -c Release
# Output: Installer/bin/Release/net8.0-windows/win-x64/publish/PWAATAccessibilityInstaller.exe
```

## Key Configuration

- **Target Framework**: `net35` (must match MelonLoader runtime)
- **GamePath**: `C:\Program Files (x86)\Steam\steamapps\common\Phoenix Wright Ace Attorney Trilogy`
- **MelonLoader References**: Use `net35` folder, not `net6`
- **UserData Config**: Runtime overrides in `$(GamePath)\UserData\AccessibilityMod\`
- **UniversalSpeech**: Requires `UniversalSpeech.dll` (32-bit) in the game directory for screen reader output
- **Logs**: MelonLoader logs to `$(GamePath)\MelonLoader\Latest.log` - use `AccessibilityMod.Logger.Msg()` for debug output

## Architecture

### Core Components

- **AccessibilityMod.Core.AccessibilityMod**: Main MelonMod entry point. Handles initialization and keyboard input via `OnUpdate()`
- **MelonLoggerAdapter**: Bridges MelonLoader logging with the MelonAccessibilityLib library
- **CoroutineRunner**: MonoBehaviour singleton for menu cursor tracking and delayed announcements

### MelonAccessibilityLib (NuGet dependency)

The following components are provided by the `MelonAccessibilityLib` NuGet package:

- **UniversalSpeechWrapper**: Low-level P/Invoke wrapper around UniversalSpeech library for text-to-speech output
- **SpeechManager**: Centralized speech output manager
- **Net35Extensions**: Polyfills for .NET 3.5 compatibility (`IsNullOrWhiteSpace`, etc.)
- **TextCleaner**: Strips formatting tags and normalizes text for screen reader output

### Keyboard Shortcuts

| Key   | Context                           | Action                                                      |
| ----- | --------------------------------- | ----------------------------------------------------------- |
| F5    | Global                            | Hot-reload config files (character names, evidence details) |
| R     | Global (except vase/court record) | Repeat last output                                          |
| I     | Global                            | Announce current state/context                              |
| [ / ] | Investigation                     | Navigate hotspots                                           |
| U     | Investigation                     | Jump to next unexamined hotspot                             |
| F1    | Investigation                     | List all hotspots                                           |
| [ / ] | Pointing mode                     | Navigate target areas                                       |
| F1    | Pointing mode                     | List all target areas                                       |
| [ / ] | Luminol mode                      | Navigate blood evidence                                     |
| [ / ] | 3D Evidence                       | Navigate examination points                                 |
| [ / ] | Fingerprint mode                  | Navigate fingerprint locations                              |
| F1    | Fingerprint mode                  | Get hint for current phase                                  |
| [ / ] | Video tape mode                   | Navigate to targets when paused                             |
| F1    | Video tape mode                   | Get hint                                                    |
| F1    | Vase puzzle                       | Get hint for current step                                   |
| F1    | Vase show (rotation)              | Get hint                                                    |
| [ / ] | Dying message                     | Navigate between dots                                       |
| F1    | Dying message                     | Get hint for spelling                                       |
| F1    | Bug sweeper                       | Announce state/hint                                         |
| F1    | Orchestra mode                    | Announce controls help                                      |
| H     | Trial (not pointing)              | Announce life gauge                                         |

### Patches (Harmony)

All patches use `[HarmonyPostfix]` (or `[HarmonyPrefix]` for capture-before-clear) to hook into game methods:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AccessMods/PhoenixWrightTrilogy](https://github.com/AccessMods/PhoenixWrightTrilogy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
