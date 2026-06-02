---
trigger: always_on
description: **Project:** NothingLess
---

# AGENTS.md — NothingLess

**Project:** NothingLess  
**Version:** 1.0.0  
**Framework:** QtQuick / Quickshell  
**Primary Languages:** QML, JavaScript, Python, Bash, Nix  
**Compositor:** Hyprland (via `axctl` abstraction)  
**Target Platforms:** Arch Linux, Fedora, NixOS  

---

## 1. Project Overview

NothingLess is a highly customizable Wayland shell built on [Quickshell](https://git.outfoxxed.me/outfoxxed/quickshell). It provides a unified desktop environment layer including a status bar, dynamic notch ("dynamic island"), app dock, dashboard, lockscreen, desktop widgets, notification popups, and an AI assistant sidebar. The shell is driven by a reactive JSON configuration system and supports multi-monitor setups via per-screen `Variants`.

The project was forked from [Ambxst](https://github.com/Axenide/Ambxst) and maintains the same upstream license. All NothingLess-specific modifications are provided under that same license.

### Key Differentiators from Upstream
- **130+ compositor settings** across 11 categories (vs. ~40 upstream)
- **Hardware-accelerated video wallpapers** via QtMultimedia + FFmpeg (instead of mpv)
- **Custom MangoHud integration** for real-time FPS display in the notch
- **Configurable rendering backend:** OpenGL (default) or Vulkan with threaded render loop
- **Ndot dot-matrix typography** and monochrome-with-red-accents design language

---

## 2. Technology Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **UI Framework** | Qt 6 (QtQuick, QtQuick.Controls, QtQuick.Effects, QtQuick.Layouts) | Rendering, animations, controls |
| **Shell Runtime** | Quickshell (`qs`) | Wayland panel/surface manager, QML engine, IPC |
| **Compositor Bridge** | `axctl` (Go binary, external repo) | Hyprland abstraction: window focus, workspace dispatch, config persistence |
| **Configuration** | JSON on disk + `Quickshell.Io.FileView` / `JsonAdapter` | Reactive, file-backed persistent config |
| **Backend Scripts** | Python 3, Bash | System monitoring, clipboard, OCR, screenshots, wallpaper thumbs |
| **Color Generation** | `matugen` | Material You color extraction from wallpapers |
| **Packaging** | Nix Flake (`flake.nix`) | Reproducible builds, NixOS module, dev shells |
| **Install Script** | `install.sh` (Bash) | Arch / Fedora dependency install, repo clone, launcher setup |

### Runtime Dependencies
- **Core:** `quickshell`, `qt6-base`, `qt6-declarative`, `qt6-wayland`, `qt6-svg`, `qt6-multimedia`, `qt6-shadertools`, `kf6-syntax-highlighting`, `kf6-breeze-icons`
- **Compositor:** `hyprland`, `axctl`
- **System:** `brightnessctl`, `grim`, `slurp`, `wl-clipboard`, `wlsunset`, `wtype`, `upower`, `power-profiles-daemon`, `NetworkManager`, `bluetooth`
- **Media:** `playerctl`, `ffmpeg`, `gpu-screen-recorder`, `wf-recorder`
- **Fonts:** `ttf-phosphor-icons`, `ttf-ndot` (custom), `ttf-roboto`, `noto-fonts`, `noto-fonts-emoji`
- **Tools:** `kitty`, `tmux`, `fuzzel`, `matugen`, `tesseract`, `zenity`, `jq`, `sqlite`
- **Python packages** (installed via pipx where applicable): script dependencies are runtime-checked

---

## 3. Project Structure

```
./
├── shell.qml                 # Entry point: ShellRoot, Variants per screen, service init
├── cli.sh                    # Launch wrapper & IPC controller (brightness, lock, install, etc.)
├── install.sh                # Distribution-aware installer (Arch, Fedora, NixOS)
├── flake.nix                 # Nix flake: packages, devShells, apps, NixOS module
├── version                   # Single-line version string (e.g., "1.0.0")
│
├── config/                   # Central configuration system
│   ├── Config.qml            # >3700 lines. Singleton. FileView + JsonAdapter persistence
│   ├── ConfigValidator.js    # Deep-merge validation against defaults
│   ├── KeybindActions.js     # Keybind action dispatch table
│   └── defaults/*.js         # 14 default blueprints: bar.js, theme.js, ai.js, compositor.js, etc.
│
├── modules/                  # All QML code organized by domain
│   ├── bar/                  # Panel widgets: clock, systray, workspaces, battery, volume
│   ├── components/           # Reusable UI primitives + GLSL shaders (55 files)
│   ├── corners/              # Rounded screen-corners overlay
│   ├── desktop/              # Desktop background + icon grid
│   ├── dock/                 # App dock (standalone or integrated)
│   ├── frame/                # Screen border / glow effect
│   ├── globals/              # GlobalStates.qml — transient runtime state (non-persistent)
│   ├── lockscreen/           # WlSessionLock + PAM authentication
│   ├── notch/                # Dynamic island UI (launcher, dashboard, notifications)
│   ├── notifications/        # Popup system + delegate + history
│   ├── services/             # 43+ backend singletons (Battery, AI, Network, AxctlService, etc.)
│   ├── shell/                # UnifiedShellPanel + ReservationWindows + OSD
│   ├── sidebar/              # AI assistant sidebar
│   ├── theme/                # Colors, Icons, Styling singletons + app config generators
│   ├── tools/                # Screenshot, screen recording, mirror, color picker
│   └── widgets/              # Complex overlays
│       ├── config/           # Standalone settings window

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leriart/NothingLess](https://github.com/leriart/NothingLess) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
