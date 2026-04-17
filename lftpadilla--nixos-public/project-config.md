---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Structure

This is a NixOS dotfiles repository using flakes for system configuration management. The main configuration lives in the `system/` directory with Home Manager integration for user-specific settings.

### Key Directories
- `system/` - Main NixOS system configuration
- `system/applications/` - Application-specific configurations (rofi, sunshine, etc.)
- `system/modules/` - Custom NixOS modules
- `system/nvim/` - Neovim configuration files
- `users/felipe/homeassistant/` - Home Assistant companion configuration

## Build Commands

### System Rebuild
```bash
# Primary rebuild command (also available as alias: rebuild)
sudo nixos-rebuild switch --flake ~/.dotfiles#default

# Test without switching (also available as alias: rebuild-test)
sudo nixos-rebuild test --flake ~/.dotfiles#default

# Build configuration only
sudo nixos-rebuild build --flake ~/.dotfiles#default

# Boot into new configuration on next reboot (also available as alias: rebuild-boot)
sudo nixos-rebuild boot --flake ~/.dotfiles#default
```

### Flake Management
```bash
# Update flake inputs (also available as alias: update-flake)
nix flake update ~/.dotfiles/system/

# Show flake info
nix flake show ~/.dotfiles/system/

# Check flake for issues
nix flake check ~/.dotfiles/system/
```

## Architecture Overview

### Configuration Files
- `flake.nix` - Defines inputs (nixpkgs, home-manager, zen-browser) and system configuration
- `configuration.nix` - Main system configuration including hardware, networking, services, users, and packages
- `home.nix` - Home Manager configuration for user 'felipe' including dotfiles, programs, and desktop settings
- `hardware-configuration.nix` - Hardware-specific configuration (auto-generated)

### Key Features Configured
- **Desktop Environment**: GNOME with X11 (Wayland disabled)
- **Power Management**: auto-cpufreq with performance/powersave profiles
- **Development Tools**: VSCode, Cursor, Git, Node.js 22, Python 3, Docker
- **Terminal Setup**: Kitty terminal with Zsh, Oh-My-Zsh, Starship prompt, Yazi file manager
- **Virtualization**: Docker, libvirtd with QEMU/KVM support
- **Remote Access**: Sunshine game streaming, Tailscale VPN, SSH
- **Applications**: Rofi launcher, various browsers (Brave, Chrome, Ungoogled Chromium)

### Home Manager Integration
Home Manager is configured as a NixOS module, managing:
- Shell configuration (Zsh with Oh-My-Zsh and Starship)
- Terminal emulator (Kitty with custom keybindings and theming)
- File manager (Yazi with custom configuration)
- Desktop environment customizations (GNOME extensions and keybindings)
- Git configuration
- Rofi application launcher with custom themes and scripts

### Rofi Configuration
Custom Rofi setup with:
- Multiple modes: applications, windows, SSH, emoji
- Custom keybindings via GNOME (Alt+n for launcher, Alt+w for windows, Alt+e for emoji)
- Custom themes and scripts in `system/applications/rofi/`
- Additional rofi scripts: brightness control, power management, system actions
- Desktop entries for easy access

### Services and Daemons
- **hacompanion**: Home Assistant companion service
- **Sunshine**: Game streaming server with hardware acceleration
- **Tailscale**: VPN service
- **auto-cpufreq**: Dynamic CPU frequency scaling
- **thermald**: Thermal management

## Development Environment

### Languages and Tools
- **Node.js**: Version 22 installed system-wide
- **Python**: Python 3 with development packages available via nix-shell
- **Git**: Configured with user details and helpful aliases
- **Docker**: Enabled with user in docker group
- **Shell**: Zsh with Oh-My-Zsh, Starship prompt, and useful aliases

### Shell Configuration
- **Terminal**: Kitty with Catppuccin Mocha theme
- **File Manager**: Yazi with custom keybindings and configuration
- **Navigation**: Zoxide for smart directory jumping
- **Aliases**: ls → eza, cd → z (zoxide), pbcopy/pbpaste for clipboard

### Tmux Configuration

Terminal multiplexer with advanced session management and custom plugins. See [docs/tmux.md](docs/tmux.md) for complete documentation.

**Quick Start:**
- Prefix: `Ctrl-Space` (primary) or `Ctrl-b` (fallback)
- Catppuccin Mocha theme with powerline separators
- TPM plugin manager for enhanced features
- Key plugins: sessionx, thumbs, floax, tmux-fzf

**Essential Keybindings:**
- `Prefix + o` - Session manager
- `Prefix + Space` - Copy hints
- `Prefix + F` - Floating terminal
- `Ctrl+h/j/k/l` - Navigate panes (vim-aware)

### Fonts
- JetBrains Mono (with Nerd Font support)
- SF Pro Display set as system default
- Fira Code, Cascadia Code, IBM Plex available

## Power Management

Integrated CPU frequency and power profile control with multiple access methods:

### GUI Access
- **GNOME Settings**: Settings → Power → Power Mode (Power Saver, Balanced, Performance)
- **GNOME Top Bar**: Power profiles shown in system menu (when power-profiles-daemon is active)
- **Extensions**: System monitor extensions show current CPU governor

### Keyboard Shortcuts
- `Super+Alt+1` - Power Saver mode (maximum battery life)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LFTPadilla) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
