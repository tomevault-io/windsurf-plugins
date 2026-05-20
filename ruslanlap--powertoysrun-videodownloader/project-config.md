---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a PowerToys Run plugin that enables video downloading from YouTube and 1000+ sites directly from the PowerToys Run launcher using yt-dlp. The plugin automatically downloads and manages yt-dlp and ffmpeg binaries.

**Technology Stack:**
- C# with .NET 9.0 (net9.0-windows10.0.22621.0)
- Windows Presentation Foundation (WPF) and Windows Forms
- Community.PowerToys.Run.Plugin.Dependencies NuGet package (v0.91.0)
- External dependencies: yt-dlp (downloaded at runtime), ffmpeg (downloaded at runtime)

**Target Platforms:** x64 and ARM64

## Build and Test Commands

### Building the Project

```bash
# Build for all platforms (x64 and ARM64)
dotnet build VideoDownloader/VideoDownloader.sln -c Release

# Build for specific platform
dotnet build VideoDownloader/VideoDownloader.sln -c Release -p:Platform=x64
dotnet build VideoDownloader/VideoDownloader.sln -c Release -p:Platform=ARM64

# Restore dependencies
dotnet restore VideoDownloader/VideoDownloader.sln
```

Build output location: `VideoDownloader/Community.PowerToys.Run.Plugin.VideoDownloader/bin/{Platform}/Release/net9.0-windows10.0.22621.0/`

### Testing

```bash
# Run all tests
dotnet test VideoDownloader/VideoDownloader.sln

# Run tests for specific project
dotnet test VideoDownloader/Community.PowerToys.Run.Plugin.VideoDownloader.UnitTests/Community.PowerToys.Run.Plugin.VideoDownloader.UnitTests.csproj
```

### Local Installation

For testing the plugin locally in PowerToys:

```bash
# Windows batch script (run from repository root)
install-local.bat
```

Or manually copy the build output to:
```
%LOCALAPPDATA%\Microsoft\PowerToys\PowerToys Run\Plugins\VideoDownloader\
```

### Packaging for Release

```bash
# Create distributable package (Windows)
pack.bat
```

## Architecture and Key Components

### Plugin Entry Point

**Main.cs** (`VideoDownloader/Community.PowerToys.Run.Plugin.VideoDownloader/Main.cs`)
- Primary plugin class implementing `IPlugin`, `IReloadable`, `IDisposable`, `ISettingProvider`
- Plugin ID: `B8F9B9F5C3E44A8B9F1F2E3D4C5B6A7B`
- Action keyword: `dl`
- Core responsibilities:
  - Query processing and result generation
  - yt-dlp and ffmpeg management (automatic download on first use)
  - Video/audio download orchestration
  - Settings persistence via JSON serialization
  - PowerShell integration for Explorer window management

### Settings Architecture

**VideoDownloaderSettings** class (Main.cs:1663-1688)
- Settings persisted to: `%LOCALAPPDATA%\Microsoft\PowerToys\PowerToys Run\Settings\Plugins\Community.PowerToys.Run.Plugin.VideoDownloader\settings.json`
- All settings exposed through PowerToys Settings UI via `AdditionalOptions` property
- Key configuration categories:
  - Download behavior (quality, format, path)
  - Audio extraction settings
  - Subtitle handling (manual, auto-generated, embedding, language preferences)
  - Filename templates with conflict resolution
  - Notification preferences
  - UI behavior (command window visibility, auto-open folder)

### Download Flow

1. **URL Validation** → `IsValidUrl()` checks for valid HTTP/HTTPS URLs
2. **Setup Check** → Verifies yt-dlp.exe and ffmpeg.exe exist in plugin directory
3. **Command Building** → `BuildYtDlpCommand()` constructs yt-dlp arguments based on settings
4. **Execution** → Two modes:
   - **Visible mode** (`ShowCommandWindow = true`): Launches CMD window via `RunYtDlpInTerminal()`
   - **Silent mode** (default): Hidden process with PowerToys notifications via `RunYtDlpWithNotifications()`
5. **Post-download** → Smart folder opening with existing window detection via PowerShell COM automation

### External Binary Management

**Automatic Setup** (`SetupPluginAsync()`, Main.cs:121-187):
- yt-dlp: Downloaded from `https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp.exe`
- ffmpeg: Downloaded and extracted from `https://github.com/BtbN/FFmpeg-Builds/releases/download/latest/ffmpeg-master-latest-win64-gpl.zip`
- Both stored directly in plugin directory
- Update mechanism for yt-dlp with backup/restore (`UpdateYtDlp()`, Main.cs:189-247)

### Advanced Features

**Subtitle Management** (Main.cs:872-942):
- Supports multiple formats: SRT, VTT, ASS, LRC, TTML, SBV, JSON3
- Language code validation and multi-language download
- Embedding subtitles in compatible video formats (MKV, MP4, WebM)
- Auto-generated subtitle fallback

**Filename Conflict Resolution** (Main.cs:450-490):
- Uses video ID or timestamp to ensure unique filenames
- Safe ASCII filenames with `--restrict-filenames` and `--windows-filenames`
- yt-dlp output templates: `%(title).100B_[%(height)sp]_[%(id)s].%(ext)s`
- Prevents overwrites with `--no-overwrites` flag

**Smart Explorer Window Management** (Main.cs:559-656):
- PowerShell COM automation to detect existing Explorer windows
- Activates existing window/tab instead of opening duplicates
- Window restoration if minimized, foreground activation via Win32 APIs

## CI/CD Pipeline

**GitHub Actions Workflow** (`.github/workflows/build-and-release.yml`):
- Triggered on: Tags matching `v*` pattern
- Matrix build: x64 and ARM64 platforms

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ruslanlap/PowerToysRun-VideoDownloader](https://github.com/ruslanlap/PowerToysRun-VideoDownloader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
