---
trigger: always_on
description: - **Warm Burnout** is a dark humor brand name, intentionally edgy -- similar to Liquid Death. It is not offensive, it is branding.
---

# Warm Burnout -- Agent Instructions

## Brand Identity -- READ THIS FIRST

- **Warm Burnout** is a dark humor brand name, intentionally edgy -- similar to Liquid Death. It is not offensive, it is branding.
- Dark humor in copy, UI text, and documentation is intentional and must be preserved as-is.
- Agents must NOT sanitize, tone-police, soften, or rewrite text for "marketing purposes" or politeness.
- This is a theme suite, not a sensitivity project. Respect the developer audience.
- When writing any user-facing text (README, descriptions, changelogs), match the existing tone: dry, direct, slightly dark.
- Do not explain or apologize for the name. Ever.

## Project Overview

Warm Burnout is a multi-platform color theme suite designed to reduce eye strain during extended coding sessions. Fully warm palette, minimal blue-spectrum emission, contrast-audited to WCAG standards. Two variants: dark and light.

## Project Structure

```
warm-burnout/
  README.md                   # Suite README -- brand, science, scores, palette
  AGENTS.md                   # This file -- suite-level agent rules
  LICENSE                     # MIT, Felipe Lima
  Cargo.toml                  # Rust test harness for all platforms
  rustfmt.toml                # 2-space indent, 120 max width
  src/lib.rs
  tests/
    common.rs                 # Shared test utilities (hex validation, color extraction)
    brand.rs                  # Brand name enforcement tests
    canonical.rs              # Cross-platform palette consistency tests
    home_assistant.rs         # Home Assistant theme validation tests
    ghostty.rs                # Ghostty theme validation tests
    starship.rs               # Starship palette validation tests
    vscode.rs                 # VS Code theme validation tests
    zed.rs                    # Zed theme validation tests
    eza.rs                    # Eza theme validation tests
    xcode.rs                  # Xcode theme validation tests
    iterm2.rs                 # iTerm2 theme validation tests
    jetbrains.rs              # JetBrains theme validation tests
    windows_terminal.rs       # Windows Terminal theme validation tests
    warp.rs                   # Warp theme validation tests
    tmux.rs                   # tmux theme validation tests
    zellij.rs                 # Zellij theme validation tests
    zsh.rs                    # Zsh theme validation tests
    obsidian.rs               # Obsidian theme validation tests
  .github/workflows/
    validate.yml              # CI: run theme validation on push/PR
    release-vscode.yml        # VS Code extension release workflow
    release-themes.yml        # All platforms release workflow (zip/attach to GH release)
  themes/
    warm-burnout.yaml          # Symlink -> ../home-assistant/warm-burnout.yaml (HACS)
  hacs.json                    # HACS manifest (Home Assistant)
  home-assistant/              # Home Assistant theme
    warm-burnout.yaml          # Theme file (dark + light modes)
    README.md                  # Home Assistant install instructions
    AGENTS.md                  # Home Assistant-specific agent rules
  vscode/                     # VS Code extension (primary, palette source of truth)
    README.md                 # VS Code install instructions
    AGENTS.md                 # VS Code-specific agent rules
    TODO.md                   # Pre-release tasks
    package.json
    themes/
      warm-burnout-dark.json
      warm-burnout-light.json
    .vscode/launch.json
  ghostty/                    # Ghostty terminal theme
    README.md                 # Ghostty install instructions
    AGENTS.md                 # Ghostty-specific agent rules
    warm-burnout-dark         # Dark theme (no extension)
    warm-burnout-light        # Light theme (no extension)
  zsh/                        # Zsh syntax highlighting + fzf themes
    README.md                 # Zsh install instructions
    AGENTS.md                 # Zsh-specific agent rules
    warm-burnout-dark.zsh-theme   # zsh-syntax-highlighting dark
    warm-burnout-light.zsh-theme  # zsh-syntax-highlighting light
    warm-burnout-dark-fzf.zsh     # fzf dark
    warm-burnout-light-fzf.zsh    # fzf light
  starship/                   # Starship prompt theme
    README.md                 # Starship install instructions
    AGENTS.md                 # Starship-specific agent rules
    warm-burnout-dark.toml    # Dark palette (standalone)
    warm-burnout-light.toml   # Light palette (standalone)
    starship.toml             # Example full config with both palettes
  zed/                        # Zed editor theme
    README.md                 # Zed install instructions
    AGENTS.md                 # Zed-specific agent rules
    extension.toml            # Extension manifest
    LICENSE                   # Required by Zed extension registry
    themes/
      warm-burnout.json       # Theme family (dark + light in one file)
  nvim/                       # Neovim colorscheme
    README.md                 # Neovim install instructions
    AGENTS.md                 # Neovim-specific agent rules
    colors/
      warm-burnout-dark.lua   # Dark variant entry point

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [felipefdl/warm-burnout](https://github.com/felipefdl/warm-burnout) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
