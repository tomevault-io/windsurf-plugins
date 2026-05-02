---
trigger: always_on
description: This file provides context to Claude Code when working in this repository.
---

# CLAUDE.md

This file provides context to Claude Code when working in this repository.

## Project Overview

**vm-curator** is a feature-rich Rust TUI application for managing QEMU/KVM virtual machines. It provides automatic VM discovery, a 5-step creation wizard with 121 pre-configured OS profiles, snapshot management, USB passthrough, 3D graphics acceleration with NVIDIA GPUs, and comprehensive OS metadata with ASCII art for 50+ operating systems.

## Repository Structure

```
.
├── README.md
├── CLAUDE.md
├── LICENSE
└── vm-curator/
    ├── Cargo.toml
    ├── assets/
    │   ├── ascii/                    # 42+ ASCII art files (.txt)
    │   └── metadata/
    │       ├── defaults.toml         # 238 OS metadata entries
    │       ├── qemu_profiles.toml    # 121 QEMU configuration profiles
    │       └── hierarchy.toml        # 14 OS family categories
    └── src/
        ├── main.rs                   # Entry point, CLI parsing (clap)
        ├── app.rs                    # Application state machine (13+ screens)
        ├── config/
        │   └── mod.rs                # User settings (~/.config/vm-curator/)
        ├── vm/
        │   ├── mod.rs
        │   ├── discovery.rs          # VM scanning and grouping
        │   ├── launch_parser.rs      # launch.sh QEMU argument extraction
        │   ├── lifecycle.rs          # VM launching, USB passthrough
        │   ├── create.rs             # VM creation, OVMF detection
        │   ├── snapshot.rs           # qemu-img snapshot operations
        │   └── qemu_config.rs        # Parsed QEMU configuration struct
        ├── metadata/
        │   ├── mod.rs
        │   ├── os_info.rs            # OS metadata (blurbs, facts, install steps)
        │   ├── qemu_profiles.rs      # Pre-configured QEMU settings per OS
        │   ├── hierarchy.rs          # OS family categorization (14 families)
        │   └── ascii_art.rs          # ASCII logo loading and caching
        ├── hardware/
        │   ├── mod.rs
        │   ├── usb.rs                # USB device enumeration (libudev + sysfs)
        │   └── passthrough.rs        # Persistent USB passthrough config
        ├── ui/
        │   ├── mod.rs                # Main render loop, event handling
        │   ├── screens/
        │   │   ├── main_menu.rs      # VM list with hierarchy (40/60 layout)
        │   │   ├── management.rs     # VM management options
        │   │   ├── create_wizard.rs  # 5-step VM creation (2000+ lines)
        │   │   ├── configuration.rs  # VM config display
        │   │   ├── settings.rs       # Application settings editor
        │   │   └── help.rs           # Keybindings reference
        │   └── widgets/
        │       ├── vm_list.rs        # Hierarchical VM list with expand/collapse
        │       └── ascii_info.rs     # ASCII art + OS info panel
        └── commands/
            ├── mod.rs
            ├── qemu_system.rs        # QEMU emulator detection, KVM checking
            └── qemu_img.rs           # Disk image and snapshot operations
```

## Tech Stack

- **Language**: Rust (edition 2021)
- **TUI Framework**: ratatui 0.30 with crossterm 0.29
- **Async Runtime**: tokio (for VM launching with error monitoring)
- **CLI**: clap 4.5 with derive feature
- **Serialization**: serde + toml 0.9 for config/metadata, serde_json for snapshots
- **Parsing**: nom 8.0 for launch.sh argument extraction
- **USB**: libudev 0.3 with sysfs fallback
- **Other**: regex 1.11 (categorization), dirs 6.0 (home paths), include_dir 0.7 (embedded assets), chrono 0.4 (dates), anyhow 1.0 (errors)

## Key Features

### VM Discovery & Organization
- Scans `~/vm-space/` for directories with `launch.sh`
- Parses QEMU arguments: emulator, memory, CPU, VGA, audio, network, disks
- Supports architectures: x86_64, i386, ppc, m68k, arm, aarch64
- Hierarchical grouping by 14 OS families with emoji icons

### VM Creation Wizard (5 Steps)
1. **Select OS**: Choose from 121 profiles (Windows, macOS, Linux, BSD, retro) or create custom
2. **Select ISO**: File browser for installation media
3. **Configure Disk**: Size in GB, qcow2 format
4. **Configure QEMU**: Memory, CPU, display, VGA, audio, network, machine type
5. **Review & Create**: Summary with auto-launch option

### Snapshot Management
- Create, list, restore, delete snapshots (qcow2 disks only)
- Visual list with timestamps and sizes
- Background operations with progress feedback

### USB Passthrough
- libudev-based enumeration with sysfs fallback
- Hub filtering, persistent configuration
- QEMU argument generation for selected devices

### 3D Graphics Acceleration
- Para-virtualized 3D with NVIDIA GPUs (virtio-vga-gl with gl=on)
- Tested on RTX-4090 with driver 590.48.01

### OS Metadata System
- 238 OS entries with display names, publishers, release dates, architectures
- Short/long descriptions, fun facts, multi-step installation guides
- User overrides: `~/.config/vm-curator/metadata/`

### ASCII Art
- 42+ embedded OS logos (Windows, DOS, Mac, Linux distros, BSD, etc.)
- User overrides: `~/.config/vm-curator/ascii/`

## CLI Commands

```bash
vm-curator                              # Launch TUI
vm-curator list                         # List all discovered VMs
vm-curator launch <name>                # Launch VM by name

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mroboff/vm-curator](https://github.com/mroboff/vm-curator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
