---
trigger: always_on
description: SyncTray is a macOS menu bar application that provides Google Drive-style background folder sync using rclone's bisync feature. It enables seamless two-way synchronization between local folders and any of rclone's 70+ supported cloud providers (Dropbox, OneDrive, Google Drive, S3, SFTP, etc.).
---

# SyncTray Development Guidelines

## Project Overview

SyncTray is a macOS menu bar application that provides Google Drive-style background folder sync using rclone's bisync feature. It enables seamless two-way synchronization between local folders and any of rclone's 70+ supported cloud providers (Dropbox, OneDrive, Google Drive, S3, SFTP, etc.).

### Key Features
- **Multi-profile support**: Configure multiple sync pairs (local folder ↔ cloud remote)
- **Three sync modes**: Two-way sync (bisync), one-way sync (upload/download), and stream (mount)
- **Background sync via launchd**: Scheduled syncs run automatically at configurable intervals
- **Real-time file monitoring**: FSEvents-based directory watching triggers syncs on local changes
- **External drive support**: Auto-detects when external drives are mounted/unmounted
- **Live progress tracking**: Parses rclone JSON logs for real-time transfer progress
- **macOS notifications**: Batch notifications for file changes with "Open Directory" action
- **Fallback remote**: Automatic failover to an alternative remote when the primary is unreachable

### Sync Modes

| Mode | rclone Command | Description |
|------|----------------|-------------|
| Two-Way Sync | `rclone bisync` | Bidirectional sync - changes on either side sync to the other |
| One-Way Upload | `rclone sync local remote` | Local is authoritative, uploads to remote |
| One-Way Download | `rclone sync remote local` | Remote is authoritative, downloads to local |
| Stream (Mount) | `rclone mount` | Virtual filesystem - files stream on-demand without local copy |

#### Mount Mode Requirements
Stream (Mount) mode requires additional setup:
1. **macFUSE**: Install via `brew install --cask macfuse` (restart required)
2. **Official rclone binary**: Homebrew's rclone doesn't support mount. Download from https://rclone.org/downloads/

```bash
# Install macFUSE
brew install --cask macfuse
# Restart Mac

# Replace Homebrew rclone with official binary
brew uninstall rclone
curl -O https://downloads.rclone.org/rclone-current-osx-arm64.zip
unzip rclone-current-osx-arm64.zip
cd rclone-*-osx-arm64
sudo cp rclone /usr/local/bin/
sudo chmod +x /usr/local/bin/rclone
```

### How It Works
1. User configures a profile: local path, rclone remote, and sync interval
2. SyncTray generates a shell script and launchd plist for scheduled syncs
3. LogWatcher monitors the sync log file for state changes and progress
4. DirectoryWatcher monitors the local folder for file changes (triggers immediate sync)
5. NotificationService batches and displays file change notifications

## Architecture

```
SyncTray/
├── Models/           # Data models and state types
├── Services/         # Business logic and background services
├── Views/            # SwiftUI views
├── Assets.xcassets/  # App icons and images
└── SyncTrayApp.swift # App entry point and AppDelegate
```

### Models/

| File | Purpose |
|------|---------|
| `SyncProfile.swift` | Profile model with sync paths, remote config, fallback remote config, computed file paths |
| `SyncState.swift` | Sync state enum, progress struct, file change model, `ActiveTransport`, `SyncLogPatterns` for log parsing |
| `RcloneLogEntry.swift` | JSON models for parsing rclone `--use-json-log` output |
| `Settings.swift` | Global app settings (debug logging toggle) |

### Services/

| File | Purpose |
|------|---------|
| `SyncManager.swift` | Central orchestrator - manages all profile states, log watchers, directory watchers |
| `ProfileStore.swift` | Persistent storage for profiles (JSON files in `~/.config/synctray/profiles/`) |
| `SyncSetupService.swift` | Generates sync scripts, launchd plists, manages agent install/uninstall |
| `LogWatcher.swift` | FSEvents + polling hybrid file watcher for rclone log files |
| `LogParser.swift` | Parses plain text and JSON log lines into typed `ParsedLogEvent` |
| `DirectoryWatcher.swift` | FSEvents-based directory monitoring with debouncing |
| `NotificationService.swift` | Batched macOS notifications with action support |
| `TelemetryService.swift` | Opt-in OTel telemetry (traces, metrics, logs) via OTLP/HTTP |

### Views/

| File | Purpose |
|------|---------|
| `MenuBarView.swift` | Menu bar dropdown with profile status, recent changes, quick actions |
| `SettingsView.swift` | Settings window with profile list and detail editor |
| `ProfileListView.swift` | Sidebar list of profiles with add/delete controls |
| `StatusHeaderView.swift` | Header showing current sync state and progress |
| `SyncProgressDetailView.swift` | Detailed per-file transfer progress during sync |
| `RecentChangesView.swift` | List of recently synced files |

## Data Flow

### Sync Monitoring Pipeline
```
launchd triggers sync script
        ↓
Script writes to log file (~/.local/log/synctray-sync-{shortId}.log)
        ↓
LogWatcher detects file changes (FSEvents + polling fallback)
        ↓
LogParser parses lines → ParsedLogEvent (syncStarted, stats, fileChange, syncCompleted, etc.)
        ↓

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mthines/sync-tray](https://github.com/mthines/sync-tray) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
