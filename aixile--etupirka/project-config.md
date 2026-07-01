---
trigger: always_on
description: **Etupirka** is a Windows desktop application for tracking play-time of visual novels (eroge). It monitors game processes, tracks focus time, and integrates with ErogameScape (Japanese game database) for game metadata.
---

# CLAUDE.md - Etupirka Project Guide

## Project Overview

**Etupirka** is a Windows desktop application for tracking play-time of visual novels (eroge). It monitors game processes, tracks focus time, and integrates with ErogameScape (Japanese game database) for game metadata.

**Version:** 0.6.0
**Target Framework:** .NET Framework 4.8
**UI Framework:** WPF with MahApps.Metro

## Quick Reference

```
Solution: Etupirka.sln
Main Project: Etupirka/Etupirka.csproj
Entry Point: App.xaml.cs → MainWindow.xaml.cs
User Database: user.db (SQLite)
Offline Game DB: data.db (SQLite)
```

## Project Structure

```
Etupirka/
├── Etupirka.sln                    # Visual Studio solution
├── Etupirka/
│   ├── App.xaml(.cs)               # Application entry, single-instance check
│   ├── MainWindow.xaml(.cs)        # Main window, process monitoring loop
│   │
│   ├── GameData/                   # Data models
│   │   ├── GameInfo.cs             # Base game metadata
│   │   ├── GameTime.cs             # Game + play-time tracking
│   │   ├── GameExecutionInfo.cs    # Process tracking, status states
│   │   ├── DisplayInfo.cs          # Monitor display settings
│   │   └── TimeData.cs             # Play-time aggregation container
│   │
│   ├── Dialog/                     # Modal dialog windows
│   │   ├── GamePropertyDialog      # Edit game details
│   │   ├── ProcessDialog           # Select game process
│   │   ├── GlobalSettingDialog     # App settings
│   │   ├── DisplaySettingsDialog   # Per-game display config
│   │   ├── PlayTimeStatisticDialog # Statistics view
│   │   ├── GameTimeGraph           # Line chart visualization
│   │   └── AddGameFromESIDDialog   # Add game by ErogameScape ID
│   │
│   ├── Views/                      # Tab pages for statistics
│   │   ├── PlayTimeWeek/Month/30Days/All  # Time period views
│   │   └── GeneralConfig/NetworkConfig/DatabaseConfig  # Settings tabs
│   │
│   ├── Control/
│   │   └── TimeControl.xaml        # Custom time display (HH:MM)
│   │
│   ├── SatoruErogeTimer/           # Legacy data structures
│   │   ├── Eroge.cs                # Simple game-time object
│   │   └── ErogeNode.cs            # Game with execution status
│   │
│   ├── DBManager.cs                # User database operations
│   ├── InformationManager.cs       # Offline ErogameScape database
│   ├── NetworkUtility.cs           # HTTP client, proxy support
│   ├── SystemUtility.cs            # Process/window utilities
│   ├── DisplaySettings.cs          # Windows display API wrapper
│   ├── ProcessInfoCache.cs         # Process path caching
│   ├── Hotkey.cs                   # Global hotkey registration
│   ├── SingleInstance.cs           # Single-instance enforcement
│   ├── StringProcessing.cs         # Title normalization (Levenshtein)
│   └── GridViewSort.cs             # ListView sorting behavior
```

## Key Technologies

| Package | Purpose |
|---------|---------|
| MahApps.Metro 1.6.5 | Metro UI styling |
| System.Data.SQLite 1.0.98.1 | SQLite database |
| HtmlAgilityPack 1.11.34 | HTML parsing for web scraping |
| Newtonsoft.Json 13.0.1 | JSON serialization |
| Hardcodet.NotifyIcon.Wpf 1.0.5 | System tray icon |
| De.TorstenMandelkow.MetroChart | Statistics charts |

## Database Schema

### User Database (user.db)

```sql
-- Game entries
CREATE TABLE games (
    uid TEXT PRIMARY KEY,    -- 16-char random ID
    title TEXT,
    brand TEXT,
    saleday TEXT,            -- yyyy-MM-dd
    esid INTEGER             -- ErogameScape ID (0 if unknown)
);

-- Daily play-time records
CREATE TABLE playtime (
    datetime TEXT,           -- yyyy-MM-dd
    game TEXT,               -- Game UID
    playtime INTEGER,        -- Seconds played
    PRIMARY KEY (datetime, game)
);

-- Lifetime game statistics
CREATE TABLE gametimeinfo (
    uid TEXT PRIMARY KEY,
    playtime INTEGER,        -- Total seconds
    firstplay TEXT,          -- First play timestamp
    lastplay TEXT            -- Last play timestamp
);

-- Execution paths
CREATE TABLE gameexecinfo (
    uid TEXT PRIMARY KEY,
    proc_neq_exec INTEGER,   -- 1 if process != executable
    procpath TEXT,           -- Monitored process path
    execpath TEXT            -- Launch executable path
);

-- Per-monitor display settings
CREATE TABLE gamedisplayinfo (
    uid TEXT,
    device_id TEXT,
    scaling INTEGER,         -- DPI scaling %
    enabled INTEGER,
    PRIMARY KEY (uid, device_id)
);
```

### Offline ErogameScape Database (data.db)

```sql
CREATE TABLE erogamescape (
    id INTEGER PRIMARY KEY,  -- ErogameScape game ID
    title TEXT,
    saleday TEXT,
    brand TEXT
);

CREATE TABLE tableinfo (
    tablename TEXT PRIMARY KEY,
    version INTEGER
);
```

## Core Concepts

### Game Status States (GameExecutionInfo)

```
NotExist  → Game executable not found
Rest      → Process not running
Unfocused → Process running but window not focused
Focused   → Window focused, time is counting
```

### Process Monitoring Loop

Located in `MainWindow.xaml.cs`:
1. Polls every 5 seconds (configurable via `monitorInterval`)
2. Checks if tracked executables are running

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Aixile/Etupirka](https://github.com/Aixile/Etupirka) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
