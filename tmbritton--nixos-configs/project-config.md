---
trigger: always_on
description: This repository contains NixOS configurations for multiple machines using Nix flakes:
---

# NixOS Configuration Repository

## Repository Overview

This repository contains NixOS configurations for multiple machines using Nix flakes:

### Machines
- **laptop**: Development laptop with power management optimizations
- **desktop**: Desktop workstation with gaming, graphic design, and NVIDIA GPU support

### Common Configuration
- **Architecture**: aarch64-linux (ARM64)
- **Desktop Environment**: GNOME with GDM
- **Primary User**: tom
- **Time Zone**: America/Chicago
- **NixOS Version**: 24.11

This is a development-focused configuration with support for multiple programming languages, development tools, and machine-specific capabilities (gaming, graphic design on desktop; power management on laptop).

## Repository Structure

### File Organization

```
.
├── flake.nix                           # Flake definition with all machine configs
├── flake.lock                          # Flake lock file (auto-generated)
├── common.nix                          # Shared configuration across all machines
├── git.nix                            # Git configuration module (shared)
├── configuration.nix                   # OLD - can be removed
├── hosts/
│   ├── laptop/
│   │   ├── configuration.nix          # Laptop-specific configuration
│   │   └── hardware-configuration.nix # Laptop hardware config (auto-generated)
│   └── desktop/
│       ├── configuration.nix          # Desktop-specific configuration
│       └── hardware-configuration.nix # Desktop hardware config (PLACEHOLDER - needs generation)
└── Claude.md                          # This file
```

### Configuration Architecture

#### Flakes-Based Multi-Machine Setup

This repository uses **Nix flakes** to manage multiple machine configurations:

1. **flake.nix**: Defines all machine configurations (laptop, desktop) and their module dependencies
2. **common.nix**: Contains configuration shared across all machines
3. **git.nix**: Git-specific shared configuration
4. **hosts/[machine]/configuration.nix**: Machine-specific configuration (hostname, packages, services)
5. **hosts/[machine]/hardware-configuration.nix**: Hardware-specific configuration (auto-generated per machine)

#### Module Loading Order

For each machine, modules are loaded in this order:
1. hosts/[machine]/configuration.nix (machine-specific)
2. hosts/[machine]/hardware-configuration.nix (hardware-specific)
3. common.nix (shared configuration)
4. git.nix (git configuration)

### Key Configuration Areas

#### Shared Configuration (common.nix)

All machines share:
- **Bootloader**: systemd-boot with EFI support
- **Networking**: NetworkManager, custom DNS (Cloudflare 1.1.1.1)
- **Desktop**: GNOME with auto-login for user "tom"
- **Audio**: PipeWire (PulseAudio disabled)
- **VM Integration**: Spice guest agent for clipboard sharing
- **Development Environment**: Full toolchain for Node.js, Go, Python, C/C++, Lua
- **Shell Aliases**: clip, rebuild, config, rg, ls
- **Neovim Auto-Setup**: Automatically clones kickstart.nvim on first boot
- **Fonts**: JetBrainsMono Nerd Font

#### Development Environment (Shared)

All machines include toolchains for:
- **Node.js**: v22 with TypeScript and typescript-language-server
- **Go**: Go compiler with gopls LSP
- **Python**: Python 3.12 with pip
- **C/C++**: gcc, gnumake, cmake, pkg-config
- **Lua**: lua with luarocks
- **General Tools**: neovim, ripgrep, fzf, direnv, tmux, tree-sitter, eza

#### Laptop-Specific Configuration (hosts/laptop/configuration.nix)

- **Hostname**: laptop
- **Power Management**: TLP enabled with AC/battery profiles
- **Battery Thresholds**: Charge between 40-80% (if hardware supports)
- **Additional Packages**: powertop, acpi
- **Services**: upower for battery monitoring

#### Desktop-Specific Configuration (hosts/desktop/configuration.nix)

- **Hostname**: desktop
- **GPU**: NVIDIA with proprietary drivers
- **Gaming**: Steam with remote play and dedicated server support, gamemode
- **Graphic Design**: Krita, Scribus, GIMP, Inkscape, Blender
- **Tablet Support**: OpenTablet driver for drawing tablets
- **Additional Tools**: Audacity, nvtop for GPU monitoring
- **OpenGL**: Full 32-bit and 64-bit support for gaming

#### Shell Configuration (common.nix)

Aliases available on all machines:
- `clip`: Copy to clipboard using xclip
- `rebuild`: Run `sudo nixos-rebuild switch --flake .`
- `config`: Edit ~/nixos-configs/ in neovim
- `rg`: ripgrep command
- `ls`: eza with icons

#### Git Configuration (git.nix)

- Default branch: main
- Pull strategy: rebase
- User: Tom Britton (britton.tm@gmail.com)
- Editor: vim
- Custom alias: `git lg` for pretty graph logs

#### SSH Configuration (common.nix)

- SSH agent enabled
- GitHub pre-configured with ~/.ssh/id_ed25519

## Development Conventions

### File Modification Guidelines

1. **Shared Configuration**: Add to `common.nix` for settings that apply to all machines
2. **Machine-Specific**: Add to `hosts/[machine]/configuration.nix` for machine-only settings
3. **DO NOT MODIFY**: Never manually edit hardware-configuration.nix files - they're auto-generated
4. **New Modules**: Create separate .nix files for substantial new functionality and import them in flake.nix
5. **Flake Structure**: All machines must be defined in flake.nix under `nixosConfigurations`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tmbritton) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
