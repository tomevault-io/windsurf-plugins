---
trigger: always_on
description: > This file provides context for AI assistants (Claude, Copilot, etc.) working in this repository. Read this before making any changes.
---

# CLAUDE.md — AstroArch AI Assistant Guide

> This file provides context for AI assistants (Claude, Copilot, etc.) working in this repository. Read this before making any changes.

---

## Project Overview

**AstroArch** is a specialized Linux distribution based on **ArchLinux (aarch64)**, tailored for astrophotography and astronomical observation. It targets Raspberry Pi 4/5, PCs, and mini-PCs. The repo is **not a software application** — it is a collection of shell scripts, configuration files, systemd services, and Zsh plugins that together define and build an OS image.

- **Current Version:** 2.0.5 (tracked in `configs/.astroarch.version`)
- **Default user:** `astronaut` / password: `astro`
- **Hostname:** `astroarch`
- **License:** MIT
- **Architecture:** aarch64 ARM
- **Live repo clone path:** `/home/astronaut/.astroarch` (the repo itself is deployed here on the live system)

---

## Repository Layout

```
astroarch/
├── astroarch_build.sh            # Main build script (entry point)
├── create_sddev_rpi5.sh          # Write image to SD card for Raspberry Pi 5
├── build-astroarch/              # Chroot and auxiliary build scripts
│   ├── AA_build_fromAA.sh        # Build a new image from an existing AstroArch install
│   ├── astroarch_build_chroot.sh # Chroot build helper
│   ├── clear-install-astroarch.* # Service/timer/script for clean reinstall
│   └── plasmasystemsettings.sh   # Plasma settings post-build helper
├── configs/                      # All system config files deployed to the live system
│   ├── .astroarch.version        # Tracks the installed version string
│   ├── .zshrc                    # Deployed as /home/astronaut/.zshrc
│   ├── config.txt                # Raspberry Pi firmware config (/boot/config.txt)
│   ├── cmdline.txt               # Kernel boot parameters
│   ├── kde_settings.conf         # SDDM display-manager config
│   ├── kwinrc                    # KDE Window Manager settings
│   ├── kdeglobals                # KDE global theme/color config
│   ├── kscreenlockerrc           # Screen locker config (disabled by default)
│   ├── smb.conf                  # Samba share config
│   ├── xorg.conf                 # X11 server config
│   ├── startwm.sh                # XRDP session startup script
│   ├── Xwrapper.config           # XRDP X wrapper config
│   ├── 20-headless.conf          # X11 headless display config
│   ├── 99-v3d.conf               # V3D GPU X11 config for RPi
│   ├── *.rules                   # udev and polkit rules
│   ├── astroarch-maintained-packages-list.txt  # Watchlist for dependency checks
│   ├── look-and-feel/astroarch/  # Custom KDE Plasma theme
│   └── layout-templates/         # Plasma panel layout templates
├── desktop/                      # .desktop launcher files
├── plugins/                      # Zsh plugins (feature extensions)
│   ├── bluetooth/bluetooth.plugin.zsh
│   ├── ftp/ftp.plugin.zsh
│   ├── gps/gps.plugin.zsh
│   └── power_max_current/power_max_current.plugin.zsh
├── scripts/                      # Utility and maintenance scripts
│   ├── update-astroarch.sh       # Thin launcher: opens Konsole and runs update-astroarch()
│   ├── astroarch-tweak-tool.zsh  # GUI tweak tool wrapper
│   ├── resize_partition.sh       # First-boot auto-resize
│   ├── aa_motd.sh                # Message of the day shown at shell login
│   ├── wallpaper.sh              # Wallpaper helper
│   ├── clone_to_usb_bootable.sh  # Clone running system to USB
│   ├── create_ap.sh              # Create a WiFi access point
│   ├── reset-brcmfmac.sh         # Reset Broadcom WiFi driver
│   ├── 2.0.5.sh                  # Version-specific migration script
│   └── ...
├── systemd/                      # Systemd service unit files
├── wallpapers/                   # System wallpapers
├── assets/icons/                 # PNG/SVG icons referenced by .desktop files
├── README.md                     # End-user documentation
├── BUILD.md                      # Build instructions
└── CHANGELOG.md                  # Version history
```

---

## Technology Stack

| Layer | Technology |
|---|---|
| Base OS | ArchLinux (aarch64) |
| Package manager | pacman + paru (AUR) |
| Custom pacman repo | `[astromatto]` at `http://astroarch.astromatto.com:9000/$arch` |
| Desktop | KDE Plasma (plasma-x11-session) |
| Display manager | SDDM (auto-login: astronaut) |
| Shell | Zsh + oh-my-zsh (theme: af-magic) |
| Remote access | TigerVNC :5900, noVNC :8080, XRDP :3389 |
| Time sync | chrony (with optional GPS/NMEA time source via SHM) |
| Astronomy | Kstars, PHD2, INDI / indi-3rdparty, astrometry.net |
| Scripting | Bash and Zsh exclusively — no Python, Node, or other runtimes in scripts |
| Hardware support | Raspberry Pi 4/5 camera, GPIO, I2C, SPI, GPS (gpsd), RTC |

---

## Development Workflows

### Making Changes

This repo is **configuration-as-code**. Changes here are deployed to a running system via:

```bash
# On the live AstroArch system:
update-astroarch   # git pull + runs version-specific migration script
```

Or by re-building a fresh image via `astroarch_build.sh`.

### Build Workflow

```
1. Start with ArchLinux ARM base image
2. Run astroarch_build.sh inside chroot

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devDucks/astroarch](https://github.com/devDucks/astroarch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
