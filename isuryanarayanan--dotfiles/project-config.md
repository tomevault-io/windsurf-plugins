---
trigger: always_on
description: Guidelines for AI agents working in this dotfiles repository.
---

# AGENTS.md

Guidelines for AI agents working in this dotfiles repository.

## Project Overview

Personal dotfiles repo providing tmux, Neovim (LazyVim), and zsh configuration.
Languages: **Bash** (setup scripts), **Lua** (Neovim config), **Zsh** (shell config), **tmux conf**.
No build system, no test suite, no CI/CD pipeline.

**This repo is the single source of truth for the development environment.**
Config files in this repo are symlinked into their system locations.
Every edit you make here is immediately live -- there is no deploy step, no copy, no sync.
Treat every change as a production change to the running environment.

## Repository Structure

```
dotfiles/
  setup.sh                  # Full bootstrap (designed for curl | bash)
  reinstall.sh              # Sync/reconcile existing install with remote
  nvim/
    setup_nvim.sh           # Symlinks ~/.config/nvim -> repo
    nvim/                   # LazyVim config (symlink target)
      init.lua              # Entry point, requires config.lazy
      stylua.toml           # StyLua formatter config
      lazy-lock.json        # Plugin lockfile (47 plugins)
      lua/config/           # lazy.lua, options.lua, keymaps.lua, autocmds.lua
      lua/plugins/          # One file per plugin/feature concern
      plugin/after/         # After-load scripts (transparency.lua)
  tmux/
    setup_tmux.sh           # Symlinks ~/.tmux.conf -> repo
    .tmux.conf              # Tmux configuration
  zsh/
    setup_zsh.sh            # Installs tools and symlinks config files
    .zshrc                  # Main shell config (zinit, plugins, aliases, vi mode)
    .zshenv                 # Universal env vars (sourced for all zsh sessions)
    .zprofile               # Login shell env (Homebrew shellenv)
    starship/
      starship.toml         # Starship prompt config
```

## Build / Lint / Test Commands

There is no build step, test suite, or CI pipeline. The project is pure configuration.

### Formatting

Lua files: format with [StyLua](https://github.com/JohnnyMorganz/StyLua) using the
config at `nvim/nvim/stylua.toml`:

```bash
# Format all Lua files
stylua nvim/nvim/

# Format a single file
stylua nvim/nvim/lua/plugins/neo-tree.lua
```

### Linting

Shell scripts: lint with [ShellCheck](https://www.shellcheck.net/):

```bash
# Lint all shell scripts
shellcheck setup.sh reinstall.sh nvim/setup_nvim.sh tmux/setup_tmux.sh zsh/setup_zsh.sh

# Lint a single script
shellcheck zsh/setup_zsh.sh
```

### Validating Changes

- **Neovim config**: Open `nvim` and confirm no errors on startup; run `:checkhealth`
- **Tmux config**: Run `tmux source-file ~/.tmux.conf` or press `prefix + r` inside tmux
- **Zsh config**: Run `source ~/.zshrc` in a running shell, or open a new terminal
- **Setup scripts**: Test with `bash -n <script>` for syntax checking before running

## Shell Script Conventions (Bash)

### Structure and Safety
- Shebang: always `#!/bin/bash`
- Use `set -e` in orchestrator scripts (`setup.sh`, `reinstall.sh`)
- Smaller helper scripts (`setup_nvim.sh`, `setup_tmux.sh`, `setup_zsh.sh`) use explicit `exit 1` instead
- Every script defines a `main()` function and calls `main "$@"` at the end

### Naming
- Global constants: `UPPER_SNAKE_CASE` (e.g., `DOTFILES_DIR`, `NVIM_SOURCE`)
- Local variables: `local lower_snake_case` (e.g., `local distro`, `local has_changes`)
- Functions: `lower_snake_case` (e.g., `install_packages_macos`, `detect_linux_distro`)

### Logging Helpers
All orchestrator scripts define these four colorized helpers using `printf` (not `echo`):
```bash
info()  { printf "\033[1;34m[info]\033[0m  %s\n" "$1"; }
ok()    { printf "\033[1;32m[ok]\033[0m    %s\n" "$1"; }
warn()  { printf "\033[1;33m[warn]\033[0m  %s\n" "$1"; }
err()   { printf "\033[1;31m[error]\033[0m %s\n" "$1"; exit 1; }
```
Use `info`/`ok`/`warn` for status messages. `err()` is always fatal (exits with 1).

### Error Handling
- `|| true` to suppress non-fatal failures
- `|| warn "..."` for degraded-but-continuing operation
- `err "..."` for fatal errors (prints message and exits)
- Redirect expected stderr with `2>/dev/null`; test commands with `>/dev/null 2>&1`

### Comments
- Section dividers: `# ── Section Name ────────────────────────`
- Step numbering in main: `# 1. Install packages`, `# 2. Setup repo`, etc.
- File headers: multi-line descriptive block after shebang

### Quoting
- Always double-quote variable expansions: `"$HOME"`, `"$DOTFILES_DIR"`, `"$(command)"`
- Single quotes only for literal strings

### Control Flow
- `case` statements for OS/distro detection
- Guard clauses with early return: `if condition; then ok "..."; return; fi`
- Symlink checks follow a 3-state pattern: is-symlink / is-file-or-dir / does-not-exist

### Self-Containment
Each script duplicates its helper functions (no shared source file). This is intentional
so that `setup.sh` works standalone via `curl | bash`.

## Lua Conventions (Neovim Config)

### Formatting
- Indentation: **2 spaces** (per `stylua.toml`). Run `stylua` before committing.
- Column width: 120 characters
- Use `-- stylua: ignore` directive when formatter output is undesirable

### File Naming
- Plugin files: `kebab-case.lua` (e.g., `all-themes.lua`, `vim-tmux-navigator.lua`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/isuryanarayanan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
