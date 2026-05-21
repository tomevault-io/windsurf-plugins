---
trigger: always_on
description: This document provides AI agents with a comprehensive understanding of this dotfiles repository, its structure, conventions, and how to work with it effectively.
---

# Dotfiles Project Knowledge Base

This document provides AI agents with a comprehensive understanding of this dotfiles repository, its structure, conventions, and how to work with it effectively.

## Project Overview

This is a personal dotfiles repository containing configuration files for development environments, terminal tools, shell environments, and various CLI/TUI applications. The project uses a cross-platform approach supporting Windows (via Scoop), macOS (via Homebrew), and Linux (via Apt-get and Pacman).

## Directory Structure

```
dotfiles/
├── application/           # Application-specific configurations
│   ├── vscode/           # VSCode settings, snippets, keybindings, custom CSS
│   ├── ghostty/          # Ghostty terminal emulator config and shaders
│   ├── kitty/            # Kitty terminal emulator config
│   ├── karabiner/        # macOS keyboard customizer
│   ├── sketchybar/       # macOS status bar
│   ├── yasb/             # Windows status bar
│   ├── tauon/            # TauonMusicBox config and themes
│   ├── singbox/          # Proxy configuration
│   └── clash-verge-rev/  # Clash Verge proxy GUI
├── shells/               # Shell configurations
│   ├── nushell/          # Nushell config, aliases, completions, themes
│   ├── zsh/              # Zsh config and aliases
│   ├── fish/             # Fish shell config
│   └── pwsh7/            # PowerShell 7 profile
├── tui_cli/              # TUI and CLI tool configurations
│   ├── yazi/             # File manager
│   ├── zellij/           # Terminal multiplexer
│   ├── lazygit/          # Git TUI
│   ├── nvim/             # Neovim config (LazyVim-based)
│   ├── mpd/              # Music Player Daemon
│   ├── rmpc/             # MPD client
│   ├── helix/            # Modal editor
│   ├── git/              # Git configuration
│   ├── bat/              # Cat clone with syntax highlighting
│   ├── btop/             # System monitor
│   ├── opencode/         # Custom OpenCode skills
│   └── (other tools)     # Various CLI tool configs
├── packages/             # Package manager configurations
│   ├── homebrew/         # Homebrew Brewfile
│   ├── scoop/            # Scoop packages
│   └── apt-get/          # Apt packages
├── docs/                 # Documentation
│   └── INSTALL.md        # Installation instructions
├── assets/               # Images and screenshots
├── install.py            # Dotbot installer script
├── install.conf.yaml     # Symlink configuration
└── README.md             # Main documentation
```

## Key Conventions

### Installation System

The project uses Dotbot for symlink management. The installation process is defined in `install.conf.yaml` which maps source files to their destination paths in the user's home directory.

- Run `./install.py` to create symlinks
- Run `./install.py --dry-run` to preview changes
- Always backup existing configurations before installation

### Platform-Specific Configurations

The `install.conf.yaml` uses conditional logic to link platform-specific configs:

- Windows-specific: Scoop, Yasb, WSL config
- macOS-specific: Karabiner, SketchyBar, Rime, Borders, VSCode paths
- Linux-specific: General configs apply

### Theme System

The project uses the Catppuccin color palette consistently across all applications:

- Catppuccin Mocha (primary)
- Catppuccin Macchiato (alternative)
- Theme files located in `application/[app]/theme/` or `tui_cli/[tool]/themes/`

### Shell Configuration

Multiple shells are supported with Nushell as the primary shell:

- Nushell: `shells/nushell/config.nu`, `shells/nushell/env.nu`
- Zsh: `shells/zsh/.zshrc`, `shells/zsh/aliasrc`
- Fish: `shells/fish/config.fish`
- PowerShell 7: `shells/pwsh7/Microsoft.PowerShell_profile.ps1`

Shell-specific aliases are organized in the `aliases/` subdirectory within each shell's folder.

## Important Files

### Configuration Files

| File | Purpose |
|------|---------|
| `install.conf.yaml` | Symlink definitions for dotbot |
| `install.py` | Dotbot installer script |
| `shells/nushell/config.nu` | Main Nushell configuration |
| `shells/nushell/env.nu` | Nushell environment variables |
| `application/vscode/settings.json` | VSCode settings |
| `application/vscode/keybindings.json` | VSCode keybindings |
| `tui_cli/opencode/skills/` | Custom OpenCode AI skills |

### Package Manager Files

| File | Purpose |
|------|---------|
| `packages/homebrew/Brewfile` | Homebrew package list |
| `packages/scoop/scoop-packages.json` | Scoop package list |
| `packages/apt-get/backup.sh` | Apt package backup script |
| `packages/apt-get/restore.sh` | Apt package restore script |

## Working with This Project

### Adding New Configurations

1. Add the configuration file to the appropriate directory under `application/`, `shells/`, or `tui_cli/`
2. Add the symlink entry to `install.conf.yaml` following the existing format
3. If platform-specific, add conditional logic using `if:` field

### Updating Package Lists

- Homebrew: Run `brew bundle dump --describe --force --file="./packages/homebrew/Brewfile"`
- Scoop: Run `scoop export > ./packages/scoop/scoop-packages.json`
- Apt: Run `apt-mark showmanual > ./packages/apt-get/apt-packages.txt`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Efterklang/dotfiles](https://github.com/Efterklang/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
