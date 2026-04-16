---
trigger: always_on
description: This is a PyQt5-based GUI application that isolates game audio for clean Steam game recording on Linux. The app allows users to select which audio sources (game audio) should be routed directly to Steam's recording input, filtering out system audio, browser audio, and other non-game sources.
---

# Steam Audio Isolator - Copilot Instructions

## Project Overview
This is a PyQt5-based GUI application that isolates game audio for clean Steam game recording on Linux. The app allows users to select which audio sources (game audio) should be routed directly to Steam's recording input, filtering out system audio, browser audio, and other non-game sources.

**Current Version:** v0.2.0  
**Repository:** https://github.com/crashman79/steam-audio-isolator  
**Status:** Active development; Flatpak-only releases (`./build.sh`, `.github/workflows/build-release.yml`)

## Current Implementation Status
- **Distribution:** Flatpak (manifest `flatpak/io.github.crashman79.steam-audio-isolator.yml`); local `./build.sh` / `./build-and-run.sh`
- PyQt5 GUI with 6 tabs (Audio Routing, Current Routes, System Info, Settings, Profiles, About)
- System tray with Show, Settings, Apply/Clear routing, Refresh, Quit
- Settings: Copy to ~/.local/bin, Add to application menu, Start when I log in, start minimized, theme, etc.
- One-time first-run prompt to install binary to ~/.local/bin when run from elsewhere
- Real-time PipeWire source detection and categorization; direct routing via pw-cli
- Profile save/load; icon cache; GitHub Actions build and release on tag push

## How to Run
1. **Flatpak (recommended):** `./build-and-run.sh` or `./build.sh` then `flatpak run io.github.crashman79.steam-audio-isolator`.
2. **From source:** `python -m steam_pipewire.main` (venv + `pip install -r requirements.txt`).

## PipeWire Configuration Analysis (Duckov Example)

### Problem Identified
Steam's default game recording captures **ALL audio** routed to the audio sink (speakers), including:
- System notifications
- Browser audio
- Any application audio playing simultaneously

### Solution Implemented
Instead of routing through the audio sink, the app creates **direct connections** between game audio nodes and Steam's recording input:

**Current (Default) Flow:**
```
Game (137) → Audio Sink (66) → Steam Recording (154)
[All audio through sink → Steam records everything]
```

**Fixed Flow (With App):**
```
Game (137) → Direct Link → Steam Recording (154)
[System audio plays through speakers, Steam only records game]
```

### Key Node IDs (from Duckov Analysis)
- Node 66: Audio Sink (speakers)
- Node 137: Duckov.exe (Stream/Output/Audio)
- Node 154: Steam (Stream/Input/Audio)
- Active Links: 118, 121 (game to sink), 139 (sink to Steam)

### Application Detection Logic (v0.1.1+)
Sources are categorized by checking in order:

**Priority 1: Communication Apps (checked BEFORE browsers)**
- Binary check: discord, slack, zoom, telegram, teams, skype, mumble, teamspeak, element, signal, whatsapp
- Name check: discord, slack, zoom, telegram, teams, skype, mumble, teamspeak, webrtc, element, signal
- **Why first?** Prevents Electron apps (Discord, Slack) from being misidentified as browsers

**Priority 2: Browsers**
- Binary check: firefox, chrome, chromium, opera, brave, edge, vivaldi, safari, epiphany, falkon, midori, qutebrowser
- Name check: firefox, chrome, chromium, opera, brave, edge, vivaldi, safari, epiphany

**Priority 3: Games**
- Wine/Proton: wine, proton, .exe in binary (1289 lines)
├── pipewire/
│   ├── __init__.py
│   ├── source_detector.py           # PipeWire node analysis & categorization
│   └── controller.py                # pw-cli interface (615 lines)
└── utils/
    ├── __init__.py
    └── config.py                    # Profile management (169 lines)

Root files:
├── CHANGELOG.md                      # Version history (Keep a Changelog format)
├── README.md                         # User documentation
├── TECHNICAL_DETAILS.md              # PipeWire routing details
├── build.sh                          # Flatpak build (install or --bundle)
├── build-and-run.sh                  # Flatpak user install + run
├── requirements.txt                  # Python dependencies
├── setup.py                          # Package configuration
├── .github/
│   ├── workflows/build-release.yml  # Automated builds on tag push
│   └── ISSUE_TEMPLATE/              # Bug report, feature request, help templateslayui
- Exclude internal nodes: echo-cancel-*, dummy-driver, freewheel-driver

**Key Detection Fixes (v0.1.1):**
- Discord: Appears as "Chromium" or "WEBRTC VoiceEngine" but binary is "Discord"
- Vivaldi: Binary is "vivaldi-bin", now correctly detected as browser

## Project Structure
```
steam_pipewire/
├── main.py                           # Application entry point
├── ui/
│   ├── __init__.py
│   └── main_window.py               # Tabbed GUI window
├── pipewire/
│   ├── __init__.py
│   ├── source_detector.py           # PipeWire node analysis
│   └── controller.py                # pw-cli interface
└── utils/
    ├── __init__.py
    └── config.py                    # Profile management
```

## Key Features Implemented
1. **Audio Source Detection**: Queries pw-dump for all PipeWire nodes
2. **Intelligent Filtering**: Excludes system nodes, only shows audio producers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/crashman79) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
