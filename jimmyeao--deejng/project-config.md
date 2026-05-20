---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DeejNG is a modern audio mixer and controller for Windows built with WPF (.NET 9), NAudio, and SkiaSharp. It provides real-time control over system and application volumes using physical sliders connected via serial (e.g., Arduino), featuring VU meters, mute controls, and persistent configuration with profile support.

**Key Technologies:**
- .NET 9 / WPF (Windows-only)
- NAudio for audio session management
- SkiaSharp for VU meter rendering
- Serial port communication (System.IO.Ports)

## Build & Run Commands

**Build the project:**
```powershell
dotnet build DeejNG.sln
```

**Build for Release:**
```powershell
dotnet build DeejNG.sln -c Release
```

**Run the application:**
```powershell
dotnet run --project DeejNG.csproj
```

**Clean build artifacts:**
```powershell
dotnet clean
```

Note: This is a Windows-only WPF application targeting .NET 9. Requires Windows with audio devices and optionally Arduino hardware for physical slider control.

## Architecture Overview

### Core Architectural Pattern

The application uses a **manager-based architecture** with manual dependency injection via `ServiceLocator` (Core/Configuration/ServiceLocator.cs). Key managers coordinate different aspects:

1. **SerialConnectionManager** - Handles USB/COM port communication with hardware sliders
2. **AppSettingsManager** - Persistent settings with fallback paths for Server OS compatibility
3. **ProfileManager** - Multiple user profiles for different scenarios (Gaming, Streaming, etc.)
4. **DeviceCacheManager** - Caches audio device information to reduce COM calls
5. **TimerCoordinator** - Coordinates all application timers (VU meters, overlays, etc.)

### Key Components

**MainWindow (MainWindow.xaml.cs)**
- Central hub coordinating all managers and services
- Owns the collection of `ChannelControl` instances (sliders)
- Manages dispatcher timers for VU meters (25ms refresh)
- Handles audio session discovery and volume application

**ChannelControl (Dialogs/ChannelControl.xaml.cs)**
- Individual slider UI component with SkiaSharp-rendered VU meter
- Can control multiple AudioTargets (apps, devices, system audio)
- Features: mute toggle, input/output device switching, smoothing
- Uses pre-calculated segment colors for performance

**AudioService (Classes/AudioService.cs)**
- Wraps NAudio API for audio session management
- Implements session caching (5-second refresh, max 15 cached apps)
- Handles "unmapped applications" - controls all apps not assigned to sliders
- Distinguishes system processes from user applications

**OverlayService (Core/Services/OverlayService.cs)**
- Manages floating overlay window (FloatingOverlay) showing volume levels
- Configurable timeout, opacity, text color, position
- Position persistence via OverlayPositionPersistenceService

### Directory Structure

```
DeejNG/
├── MainWindow.xaml(.cs)     # Main application window
├── App.xaml(.cs)            # Application entry point
├── Classes/                 # Core utilities
│   ├── AppSettings.cs       # Settings model
│   ├── AudioService.cs      # Audio session management
│   └── AudioUtilities.cs    # Audio helper functions
├── Models/                  # Data models
│   ├── AudioTarget.cs       # Represents controllable audio target
│   ├── Profile.cs           # User profile with settings
│   ├── ThemeOption.cs       # Theme metadata
│   ├── ButtonAction.cs      # Button action enum
│   ├── ButtonMapping.cs     # Button configuration model
│   └── ButtonIndicatorViewModel.cs  # Button UI state
├── Services/                # Manager classes
│   ├── AppSettingsManager.cs       # Settings persistence
│   ├── ProfileManager.cs           # Profile management
│   ├── SerialConnectionManager.cs  # Serial communication
│   ├── DeviceCacheManager.cs       # Audio device caching
│   ├── TimerCoordinator.cs         # Timer lifecycle
│   └── ButtonActionHandler.cs      # Button action execution
├── Dialogs/                 # UI components and dialogs
│   ├── ChannelControl.xaml(.cs)    # Slider component
│   ├── SessionPickerDialog.xaml    # App picker
│   ├── SettingsWindow.xaml         # Settings UI
│   └── ...
├── Views/                   # Additional windows
│   └── FloatingOverlay.xaml(.cs)   # Transparent overlay
├── Core/
│   ├── Configuration/       # ServiceLocator for DI
│   ├── Interfaces/          # Service interfaces
│   ├── Services/            # Overlay, power management services
│   └── Helpers/             # Screen position utilities
├── Infrastructure/
│   └── System/              # System integration (startup, registry)
└── Themes/                  # XAML theme files
```

### Audio Target Types

An `AudioTarget` can represent:
1. **System Audio** - Master volume control
2. **Application** - Specific app by executable name (e.g., "Spotify.exe")
3. **Input Device** - Microphone/recording device
4. **Output Device** - Speakers/headphones
5. **Unmapped Applications** - All apps not assigned to any slider

Each slider (ChannelControl) can control multiple targets simultaneously.

### Serial Protocol


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jimmyeao/DeejNG](https://github.com/jimmyeao/DeejNG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
