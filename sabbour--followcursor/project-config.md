---
trigger: always_on
description: > Additional instruction files in `.github/instructions/` cover issue implementation conventions and release/versioning.
---

# FollowCursor — Copilot Agent Instructions

> Additional instruction files in `.github/instructions/` cover issue implementation conventions and release/versioning.

## Project Overview

FollowCursor is a **Windows screen recorder** with cinematic cursor-following zoom. It captures screen or window content, tracks mouse/keyboard/click activity, and exports polished MP4 or GIF files where the camera smoothly follows and zooms into the user's cursor movements.

**Target audience**: People creating tutorials, demos, and product walkthroughs.

## Tech Stack

| Component | Technology | Notes |
|-----------|-----------|-------|
| Language | Python 3.13 | Windows only |
| UI Framework | PySide6 (Qt 6) | Frameless window, custom dark theme |
| Screen Capture | dxcam (DXGI) + mss fallback | Hardware-accelerated monitor capture |
| Window Capture | Win32 PrintWindow (ctypes) | Per-window capture without bleed-through |
| Video Export | ffmpeg via imageio-ffmpeg | H.264 MP4 or GIF piped via stdin |
| Image Processing | OpenCV (cv2) + NumPy | Frame manipulation, thumbnails, cursor rendering |
| Input Tracking | Win32 Hooks (ctypes) | WH_MOUSE_LL, WH_KEYBOARD_LL via WINFUNCTYPE |
| Build | PyInstaller | Single-folder .exe distribution |
| Installer | MSIX | Signed Windows package via Azure Trusted Signing |
| CI | GitHub Actions | Windows runner, artifact upload |
| AI Features | azure-ai-inference | Optional AI zoom analysis, TTS voiceover via Azure AI Foundry |

## Repository Structure

```
followcursor/                    ← repo root
├── .github/
│   ├── copilot-instructions.md  ← This file (general context)
│   ├── instructions/            ← Domain-specific Copilot instructions
│   ├── prompts/                 ← Copilot prompt files (e.g. fix-errors)
│   └── workflows/build.yml     ← GitHub Actions CI
│                  auto-rebase.yml ← Auto-rebase open PRs on push to main
├── .vscode/                     ← VS Code config (launch, tasks, settings)
├── followcursor/                ← Python project root
│   ├── main.py                  ← Entry point
│   ├── requirements.txt         ← Python dependencies
│   ├── generate_msix_assets.py  ← Generate MSIX tile PNGs from app icon
│   ├── followcursor.ico         ← App icon
│   ├── msix/                    ← MSIX packaging files
│   │   ├── AppxManifest.xml     ← Package manifest template
│   │   └── Assets/              ← Generated tile PNGs (gitignored)
│   ├── scripts/                 ← Build & infra PowerShell scripts
│   │   ├── Build-App.ps1        ← PyInstaller build script
│   │   ├── Start-Dev.ps1        ← Dev setup & launch script
│   │   ├── Build-Msix.ps1      ← MSIX packaging + signing (local PFX or Azure)
│   │   ├── Publish-SignedMsix.ps1 ← Locally sign, verify, and publish release MSIX
│   │   └── Setup-AzureSigning.ps1 ← Provision local Azure Trusted Signing resources
│   └── app/                     ← Application package
│       ├── main_window.py       ← Central coordinator
│       ├── models.py            ← Data classes
│       ├── screen_recorder.py   ← Capture engine (monitor + window modes)
│       ├── video_exporter.py    ← H.264 MP4 / GIF export with zoom/cursor/bezel
│       ├── compositor.py        ← QPainter compositing (frame + background)
│       ├── zoom_engine.py       ← Ease-out keyframe interpolation
│       ├── activity_analyzer.py ← Auto-zoom from activity bursts
│       ├── ai_service.py        ← AI zoom+pan analysis, TTS voiceover (Azure AI Foundry)
│       ├── mouse_tracker.py     ← QTimer cursor polling (60 Hz)
│       ├── keyboard_tracker.py  ← Win32 keyboard hook
│       ├── click_tracker.py     ← Win32 mouse click hook
│       ├── cursor_renderer.py   ← Arrow cursor rendering in export
│       ├── global_hotkeys.py    ← Ctrl+Shift+=/- zoom hotkeys
│       ├── window_utils.py      ← Win32 window enum & PrintWindow
│       ├── backgrounds.py       ← 84 background presets (solids + gradients + wavy patterns)
│       ├── frames.py            ← 5 device frame presets
│       ├── project_file.py      ← .fcproj ZIP save/load (with voiceover audio)
│       ├── icon.py              ← QPainter-generated app icon + .ico
│       ├── theme.py             ← DARK_THEME QSS stylesheet
│       └── widgets/
│           ├── title_bar.py
│           ├── source_picker.py ← Tabs: Screens / Windows
│           ├── preview_widget.py
│           ├── timeline_widget.py
│           ├── editor_panel.py
│           ├── countdown_overlay.py
│           ├── processing_overlay.py
│           └── recording_border.py
```

## Development Workflow

- **Run**: `scripts\Start-Dev.ps1` or press `F5` in VS Code
- **Build**: `scripts\Build-App.ps1` or press `Ctrl+Shift+B`
- **Test**: Execute the **Run Tests** VS Code task (do not run pytest manually in a terminal)
- **Debug**: F5 launches with debugpy attached
- VS Code automation terminals use `cmd.exe` (not WSL)

### Branching


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sabbour/followcursor](https://github.com/sabbour/followcursor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
