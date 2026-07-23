---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Agent safety

- Never close, kill, restart, or log out user applications/processes (for example Chrome, Teams, browsers, portals, Hyprland/session services) without explicit user confirmation immediately before the command.
- Treat `kill`, `killall`, `pkill`, `systemctl --user restart`, session reloads, and commands that may interrupt calls/screen sharing as risky even when they look harmless.
- For Chrome/Chromium theme debugging, do not edit browser `Preferences` or close the browser. Validate via `gsettings` and `org.freedesktop.portal.Settings` instead.

## Agent skills

### Issue tracker

Issues are tracked in GitHub Issues for `mswell/dotfiles`, using the `gh` CLI. See `docs/agents/issue-tracker.md`.

### Triage labels

Triage uses the default canonical labels. See `docs/agents/triage-labels.md`.

### Domain docs

This repo uses a single-context domain docs layout. See `docs/agents/domain.md`.

## Overview

This is a comprehensive dotfiles management system for automated Linux development environment setup. It supports multiple Linux distributions (Ubuntu, Arch Linux), with Hyprland as the primary desktop target, and includes security/pentesting tools for bug bounty hunting and security research.

## Core Commands

### Initial Setup
```bash
./install.sh  # Interactive menu with 8 setup options
```

**Menu Options:**
- `[1]` - Ubuntu/Debian VPS (servers, web development)
- `[2]` - Arch Linux VPS (servers, CLI-only, no GUI)
- `[3]` - Arch Linux + Hyprland (modern desktop, Wayland)
- `[4]` - Install Hacktools (security testing, CTF, bug bounty)
- `[5]` - Install Dev Environment (mise: Python + Node.js + pnpm)
- `[6]` - Arch Linux WSL (Windows Subsystem for Linux)
- `[7]` - Claude for Bug Bounty (Skills + Agents + Caido AI)
- `[8]` - Install Pi Coding Agent + Restore Pi Config

### Dependency Chain

**Critical:** Scripts must be run in the correct order:
1. Run distro-specific setup first (option 1, 2, 3, or 6) - installs system dependencies
2. Run dev environment installation (option 5) - sets up Python + Node.js + pnpm via mise
3. Run hacktools installation (option 4) - installs security tools

**Do not run devenv or hacktools before system setup** - they depend on system packages being installed first.

## Architecture

### Shared Libraries (`setup/lib/`)

Common functionality is extracted into shared libraries to eliminate duplication:

- `common.sh` - Universal library (all variants): DOTFILES detection, colors, `source_script()`
- `arch.sh` - Arch-specific library: `install_pacman()`, `install_yay()`, `ensure_yay()`, `arch_base_setup()`, `setup_bat_theme()`, `setup_nvim_dir()`, `install_fonts()`
- `shell_utils.sh` - Shell management: `change_shell_to_zsh()`
- `logging.sh` - Structured logging system (used by `install.sh`)
- `preflight.sh` - Pre-flight validation checks (used by `install.sh`)

### Modular Design

Each distribution has its own setup directory with specialized scripts:

**Ubuntu** (`setup/ubuntu/`):
- `setup.sh` - Orchestrator that sources all other scripts
- `base.sh` - System dependencies (build-essential, git, curl, etc.)
- `devel.sh` - Development tools (neovim, virtualenvwrapper)
- `apps.sh` - Applications (Docker, cargo tools, etc.)

**Arch Linux** (`setup/ArchHypr/`, `setup/ArchWSL/`, `setup/ArchVPS/`):
- `setup.sh` - Orchestrator (sources `lib/common.sh` for `source_script()`)
- `base.sh` - Base system via `arch_base_setup()` + variant-specific extras
- `apps.sh` - Variant-specific packages (uses shared `install_yay()`/`install_pacman()`)
- `fonts.sh` - Font installation via shared `install_fonts()` (Hyprland flow)

**Shared scripts** (used by all variants):
- `setup/terminal.sh` - TPM installation + shell change to zsh
- `setup/copy_dots.sh` - Copies config files to home directory

### Environment Variables (config/zsh/env.zsh)

**Central configuration file** - single source of truth for all paths:
```bash
TOOLS_PATH="$HOME/Tools"              # Security tools installation
LISTS_PATH="$HOME/Lists"              # Wordlists and SecLists
RECON_PATH="$HOME/Recon"              # Bug bounty recon workspace
DOTFILES_PATH="$HOME/Projects/dotfiles"
CUSTOM_NUCLEI_TEMPLATES_PATH="$DOTFILES_PATH/custom_nuclei_templates"
```

**Important:** All scripts source this file. When modifying paths, update env.zsh only.

### Security Tools (setup/install_hacktools.sh)

Installs bug bounty and security testing tools:
- **ProjectDiscovery tools** via pdtm: nuclei, httpx, subfinder, naabu, dnsx, katana, etc.
- **Go tools**: ffuf, amass, gf, qsreplace, waybackurls, gospider, puredns, etc.
- **Python tools**: dirsearch, SecretFinder, waymore, xnLinkFinder
- **Wordlists**: Automatically downloads SecLists wordlists to `$LISTS_PATH`

Sources `config/zsh/env.zsh` for all path variables.

### Development Environment (setup/devenv_install.sh)

Uses **mise** (ex-rtx) - unified version manager replacing pyenv, asdf, nvm:

- Installs mise via official installer
- Installs latest Python, Node.js, and pnpm versions reported by `mise latest` at install time

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mswell/dotfiles](https://github.com/mswell/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
