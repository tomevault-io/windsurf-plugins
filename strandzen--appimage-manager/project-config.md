---
trigger: always_on
description: This file is the authoritative reference for working on AppImage Manager.
---

# GEMINI.md

This file is the authoritative reference for working on AppImage Manager.

## Project Overview

> **Disclaimer:** This project and its author are in no way affiliated with KDE or the KDE e.V. organization. While the project's goal is to provide a native-like experience on the Plasma desktop, it is an independent, community-driven project authored by the user. It should not pretend to be an official KDE product (e.g., `org.kde.*`).

**AppImage Manager** is a lightweight KDE Plasma 6 utility for installing, managing, and removing AppImage files. It integrates with Dolphin via a right-click context menu plugin and provides a standalone dashboard for browsing all installed AppImages.

**Distribution target:** KDE Store and AUR.  
**Philosophy:** Simplicity and efficiency. No unnecessary abstractions. Every feature must justify its existence.  
**Stack:** C++20, Qt6, KDE Frameworks 6, Kirigami (Plasma 6 era), QML.  
**License:** GPL-2.0-or-later.

### Roadmap (planned, not yet implemented)

- **Signature verification** — verify AppImage signatures before install.

---

## Build Commands

```bash
# Configure + build (dev — Debug, installs to /usr, exports compile_commands.json)
cmake --preset dev
cmake --build --preset dev

# Install to /usr (required for Dolphin plugin discovery)
sudo cmake --install build/dev

# Reload Dolphin plugin without logout
kquitapp6 dolphin && dolphin &

# Release build
cmake --preset release
cmake --build --preset release
sudo cmake --install build/release

# Clean rebuild
rm -rf build/ && cmake --preset dev && cmake --build --preset dev
```

Build output: `build/<presetName>/`. No test suite — validation is manual (see Testing).

---

## Architecture — Build Targets

| Target | Type | Description |
|--------|------|-------------|
| `appimagemanager_core` | Object library | Pure C++, zero Qt Quick dependency. Reader, manager, info struct, logging |
| `appimagemanager_qml` | Shared lib + QML module `appimagemanager` | All GUI backend classes + QML files |
| `appimagemanager` (plugin `.so`) | `KAbstractFileItemActionPlugin` | Dolphin right-click plugin, auto-discovered via JSON metadata |
| `appimagemanager_bin` → `appimagemanager` | Executable | Service menu launcher + CLI. No args → dashboard. With file arg → manage window |

Both the plugin `.so` and the binary link `appimagemanager_qml` — identical logic, two entry points.

---

## Class Map

### Core (`src/core/`) — no Qt Quick

| Class | Files | Role |
|-------|-------|------|
| `AppImageInfo` | `appimageinfo.h` | Value struct: `originalName`, `cleanName`, `appId`, `appName`, `version`, `categories`, `comment`, `description`, `execArgs`, `fileSize`, `iconData`, `iconExt`, `updateInfo`, `isValid`. `description` = AppStream XML content; falls back to `comment` (`.desktop Comment=`) if XML absent |
| `AppImageReader` | `appimagereader.h/.cpp` | **BLOCKING** extractor. Requires `libappimage` (in-process SquashFS, no FUSE/subprocess). Reads `.desktop` via `KDesktopFile`, then AppStream XML from `usr/share/metainfo/*.appdata.xml` or `usr/share/appdata/*.metainfo.xml` via `QXmlStreamReader`. **Always call via `QtConcurrent::run`** |
| `AppImageCache` | `appimagecache.h/.cpp` | Thread-safe on-disk cache (`QSettings` INI, keyed by MD5(path)+mtime). **Version 2** — serializes `comment` and `description`. Entries with `cacheVersion < 2` auto-invalidate. |
| `AppImageManager` | `appimagemanager.h/.cpp` | File operations: `installAppImage()` (KIO::CopyJob + chmod +x), `createDesktopLink()`, `removeDesktopLink()`, `isDesktopLinkEnabled()`, `findCorpses()` (blocking), `removeItems()` (KIO::trash). Also `rebuildSycoca()` |
| `AppSettings` | `appsettings.h/.cpp` | QML singleton (`AppSettings`). KSharedConfig → `appimagemanagerrc`. Properties: `applicationsPath`, `showDisclaimer`, `showNotifications`, `updateFrequency`, `customUpdateDays`, `manageIconSize`, `watchDownloads` (controls both daemon + dashboard download detection), `showInstallBox`. Setter validates path via `QDir::mkpath()`, emits `applicationsPathError(msg)` on failure |
| `GitHubReleaseChecker` | `githubreleasechecker.h/.cpp` | Parses `gh-releases-zsync\|owner\|repo\|...` update info, hits GitHub Releases API, emits `updateAvailable(newVersion, zsyncUrl)`, `upToDate()`, or `failed()`. Used by both `UpdateDaemon` and `AppImageListModel` — single source of truth for GitHub update logic |
| `UpdateDaemon` | `updatedaemon.h/.cpp` | Background update checker. Launched via `--daemon` CLI flag (autostart desktop file installs to `$KDE_INSTALL_AUTOSTARTDIR`). Scans `applicationsPath` hourly, uses `GitHubReleaseChecker` for GitHub updates, fires `KNotification` on update found. Also watches `~/Downloads` for new AppImages. Registers D-Bus name `io.github.appimagemanager.Daemon` on start so the dashboard can skip duplicate download notifications. Uses `Qt6::Network` |

### GUI (`src/gui/`) — Qt Quick dependency

| Class | Files | Role |
|-------|-------|------|
| `AppImageBackend` | `appimagebackend.h/.cpp` | QML context property `backend`. Owns `AppImageInfo` + all mutable install state. Exposes metadata + ops as Q_PROPERTYs and slots. Holds `CorpseModel`. Emits `uninstallFinished` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [strandzen/AppImage-Manager](https://github.com/strandzen/AppImage-Manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
