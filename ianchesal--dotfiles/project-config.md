---
trigger: always_on
description: This file provides guidance to AI agents working on this repository.
---

# Dotfiles

This file provides guidance to AI agents working on this repository.

## Target Platforms

- macOS (zsh/Kitty), Debian Linux, and WSL2
- Changes should be portable across all three unless explicitly platform-specific

## Build/Test/Lint Commands

- Install dotfiles: `rake all`
- Install specific configuration: `rake <tool>` (e.g., `rake tmux`, `rake nvim`, `rake zsh`, `rake git`, `rake kitty`, `rake ohmyposh`)
- Remove customizations: `rake clean`
- List all rake tasks: `rake -T`
- Run Rubocop checks: `rake rubocop:check`
- Auto-correct Rubocop issues: `rake rubocop:auto_correct`
- Update configurations: `rake update`
- Update Neovim plugins: `rake nvim:update`
- Check and commit Neovim dependency updates: `rake nvim:commit`
- Check for Oh My Posh updates: `rake ohmyposh:check_update`
- Update Oh My Posh: `rake ohmyposh:update`
- Reload tmux config in all sessions: `rake tmux:reload`
- Update zsh and plugins: `rake zsh:update`
- Update Homebrew packages: `rake brew:update`
- Update Python packages: `rake python:update`
- Update yt-dlp: `rake ytdlp:update`
- Update gcloud components: `rake gcloud:update`

## Code Style Guidelines

- Ruby: Follow Rubocop guidelines in `./rubocop/rubocop.yml`
- Line length limit: 160 characters
- Indentation: 2 spaces
- Prefer single quotes for strings unless interpolation is needed
- Use snake_case for methods and variables
- Use frozen_string_literal pragmas in Ruby files
- Shell scripts should use proper error handling
- Document rake tasks with descriptions
- Method naming: Use descriptive names that reflect functionality
- File organization: Group related files by tool/function in separate directories

## AeroSpace Configuration

- A tiling window manager that uses the tree paradigm made popular by the [i3 window manager](https://i3wm.org/)
- Located in `.config/aerospace` following XDG directory structure
- Uses a TOML format for configuration
- The guide for how to configure AeroSpace can be found [here](https://nikitabobko.github.io/AeroSpace/guide)
- The Github project for AeroSpace can be found [here](https://github.com/nikitabobko/AeroSpace)

## Tmux Configuration

- Deployed to `~/.config/tmux` following XDG directory structure (the repo's `./tmux/` dir is symlinked there)
- Main configuration in `tmux.conf`, theme in `theme.conf`
- Uses TPM (Tmux Plugin Manager) for plugins, installed to `~/.config/tmux/plugins/`
- VHS Era theme with powerline-style status bar segments and double-arrow separators
- Custom helper scripts (e.g., `git-aware-popup.sh`) live in the `./tmux/` directory
- Plugin-specific configuration is grouped in labeled sections within `tmux.conf` (not inline with plugin declarations)
- Status bar uses the VHS Era color palette defined in the Oh My Posh section
- TPM bootstrap (`run '~/.config/tmux/plugins/tpm/tpm'`) must always be the last line in `tmux.conf`

## Neovim Configuration

- Based on [LazyVim](https://www.lazyvim.org/) with custom plugins and configurations
- Some LazyVim features are enabled via the `./nvim/lazyvim.json`
- Lua files should follow stylua.toml format (2 space indent, 120 column width)
- Plugin configuration belongs in `./nvim/lua/plugins/*.lua` files
- Custom keymaps should be defined in `./nvim/lua/config/keymaps.lua`
- Custom autocmd config should be defined in `./nvim/lua/config/autocmds.lua`
- Use which-key for documenting keybindings with descriptive labels
- Respect existing plugin organization and structure
- Requires Neovim 0.10+ for compatibility

## Zsh Configuration

- Organized using XDG directory structure with configs in `.config/zsh`
- Zsh files are modular and stored in `./zsh/zshrc.d/` directory
- Using `zdharma-continuum/zinit` for zsh plugin support
- Custom functions go in `./zsh/functions/` directory with one function per file
- Custom completions go in `completions/` directory following zsh-completions format
- Aliases go in the `./zsh/zshrc.d/04-aliases.zsh` file
- Naming convention for zshrc.d files: numeric prefix for load order (e.g., `02-functions.zsh`) (but only if load order is strictly required)
- Global aliases use suffix format (e.g., `alias -g G='| grep -E'`)
- History settings: large history size, ignore duplicates, share across sessions
- Use descriptive comments for functions and aliases
- Follow existing patterns for tool-specific configurations (each tool has its own file)
- NEVER recommend the forgit zsh plugin -- I hate it

## Git Configuration

- Located in `.config/git` following XDG directory structure
- Main configuration in `config` file with local overrides in `local` file
- Use descriptive git aliases that enhance workflow speed
- Git hooks stored in `hooks/` directory (pre-push hook prevents pushing fixup commits)
- GitHub CLI configuration in `gh/config.yml` with helpful aliases
- GitHub Dashboard config in `gh-dash/config.yml` for PR management
- Delta used for enhanced diffs with side-by-side display
- Conventions for commit messages: no fixup commits in pushed branches
- Git workflow relies heavily on custom aliases and integrations
- Repository configuration uses a rebase workflow with `autosetuprebase = always`

## Kitty Configuration

- Located in `.config/kitty` following XDG directory structure

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ianchesal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
