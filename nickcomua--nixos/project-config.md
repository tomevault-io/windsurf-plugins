---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Repository Overview

This is a multi-system Nix configuration built using the flake-parts architecture pattern from [tsandrini/flake-parts-builder](https://github.com/tsandrini/flake-parts-builder/). It manages NixOS (with Hyprland), macOS (nix-darwin), and Raspberry Pi deployments with comprehensive home-manager integration.

## Essential Commands

### System Management

```bash
# --- NixOS (x86_64-linux) ---
# Rebuild and switch to new configuration
sudo nixos-rebuild switch --flake ~/.config/nixos/#nixos

# Build without applying (test for errors)
nixos-rebuild build --flake ~/.config/nixos/#nixos

# --- macOS / nix-darwin (aarch64-darwin) ---
# Rebuild and switch to new configuration
darwin-rebuild switch --flake ~/.config/nixos/#Mykolas-MacBook-Pro

# Build without applying
darwin-rebuild build --flake ~/.config/nixos/#Mykolas-MacBook-Pro

# --- Flake management ---
# Update all flake inputs
nix flake update

# Update a specific input
nix flake update <input-name>

# Show flake outputs
nix flake show

# Check flake syntax and linting
nix flake check

# Format nix files
nix fmt
```

### Development

```bash
# Enter devenv shell (defined in flake-parts/devenv)
nix develop
```

### Version Control

**This repository uses Jujutsu (jj) instead of git.** Jujutsu is a Git-compatible VCS that provides a better user experience.

```bash
# Check status
jj status

# Create a new commit
jj commit -m "Description of changes"

# Push changes to remote
jj git push

# Typical workflow after making changes
jj commit -m "Add feature X"
jj git push
```

## Architecture

### Flake-Parts Module Loading System

The repository uses a custom recursive module loader (`flake-parts/_bootstrap.nix`) that automatically discovers and imports all `.nix` files in `flake-parts/`. Key behaviors:

1. **Files/directories starting with `_` or `.git` are ignored**
2. **If `myDir/default.nix` exists, only that file is loaded (children are not recursed)**
3. **All other `.nix` files are automatically imported**

This means you can organize modules arbitrarily and they'll be discovered automatically - no need to manually add imports to a central file.

### Directory Structure

```text
flake-parts/
├── _bootstrap.nix           # Module loading system (not auto-imported due to _ prefix)
├── systems.nix              # Supported systems: x86_64-linux, aarch64-linux, aarch64-darwin
├── formatter.nix            # nix fmt configuration
├── devenv/                  # Development environment config
│   ├── default.nix
│   └── dev.nix
├── hosts/                   # Host configurations
│   ├── default.nix         # mkHost/mkDarwinHost functions and configurations
│   ├── nixos/              # Main NixOS host (x86_64-linux)
│   │   ├── default.nix     # Host-level NixOS config
│   │   ├── nick.nix        # User home-manager config (imported directly)
│   │   ├── hardware-configuration.nix
│   │   └── apfs.nix        # APFS filesystem support
│   ├── alta/               # Raspberry Pi (aarch64-linux)
│   │   ├── default.nix
│   │   ├── configuration.nix
│   │   └── hardware-configuration.nix
│   └── darwin/             # macOS hosts
│       └── mykolas-macbook-pro/
│           ├── default.nix  # Darwin system config (Homebrew, system defaults, users)
│           ├── nick.nix     # nick user home-manager config
│           └── nickp.nix   # nickp user home-manager config
├── homes/                   # Home-manager standalone configs (currently unused)
└── modules/
    ├── _shared-nix.nix      # Shared Nix cache/substituter settings (excluded from auto-load)
    ├── _programs/           # Program modules (excluded from auto-load)
    │   ├── horse-browser/
    │   ├── librepods/
    │   └── whisper-transcribe/
    ├── darwin/              # Darwin system-level modules (imported directly by darwin hosts)
    │   ├── default.nix      # Stops auto-loader recursion
    │   └── monitoring.nix
    ├── nixos/               # System-level NixOS modules
    │   ├── default.nix
    │   └── monitoring.nix
    └── home-manager/
        ├── default.nix      # Exports homeModules via importApply
        ├── shared/          # Cross-platform (zsh, packages, clawdbot)
        │   ├── default.nix
        │   ├── zsh.nix
        │   ├── packages.nix
        │   └── clawdbot.nix
        ├── linux/           # Linux-specific modules
        │   └── common.nix
        ├── darwin/          # macOS-specific modules
        │   └── common.nix
        ├── wayland/         # Wayland/Hyprland ecosystem
        │   ├── default.nix
        │   ├── options.nix  # Centralized wayland option definitions
        │   ├── hyprland/
        │   ├── noctalia/    # Noctalia desktop shell (bar, launcher, control center)
        │   ├── hyprlock/
        │   ├── hypridle/
        │   ├── hyprpaper/
        │   └── satty/
        └── services/        # User services (systemd --user)
            └── activitywatch/
```

### Home-Manager Integration

#### NixOS (nixos host)

Uses **NixOS integration mode** (not standalone):

1. `hosts/default.nix` includes `home-manager.nixosModules.home-manager` when `withHomeManager = true`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nickcomua) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
