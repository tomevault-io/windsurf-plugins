---
trigger: always_on
description: <!-- Canonical source: AGENTS.md — keep this file in sync -->
---

<!-- Canonical source: AGENTS.md — keep this file in sync -->
# AGENTS.md — dotfiles

> Personal dotfiles managed by [chezmoi](https://www.chezmoi.io/). Bash-based CLI utilities use the [options.bash](https://github.com/uhop/options.bash) library for option parsing and rich terminal output. Targets Ubuntu (Debian) and macOS.

For project structure and module layout see [ARCHITECTURE.md](./ARCHITECTURE.md).
For a compact reference of all utilities see [llms.txt](./llms.txt).

## Setup

This repo is a chezmoi source directory. Apply with:

```bash
brew install chezmoi
chezmoi init --apply uhop
```

On first run, `run_onchange_before_install-packages.sh.tmpl` installs system packages via `apt`/`brew`, clones `options.bash` as a sparse worktree into `~/.local/share/libs/scripts`, installs `nvm`, `bun`, `tmux` plugins, and configures `doas`.

## Project structure

```
dotfiles/                              # chezmoi source directory
├── .chezmoi.toml.tmpl                 # chezmoi config template (OS detection, GUI flag)
├── .chezmoiignore                     # files excluded from deployment
├── dot_bashrc                         # → ~/.bashrc (shell init, PATH, completions)
├── dot_bash_aliases                   # → ~/.bash_aliases (aliases, functions)
├── dot_bash_profile                   # → ~/.bash_profile
├── dot_profile                        # → ~/.profile
├── dot_gitconfig                      # → ~/.gitconfig
├── dot_inputrc                        # → ~/.inputrc
├── dot_ackrc                          # → ~/.ackrc
├── dot_config/                        # → ~/.config/ (app configs)
│   ├── alacritty/
│   ├── cheat/
│   ├── fastfetch/
│   ├── ghostty/
│   ├── git/
│   ├── kitty/
│   ├── micro/
│   ├── nano/
│   └── tmux/
├── private_dot_local/
│   ├── bin/                           # → ~/.local/bin/ (CLI utilities)
│   │   ├── executable_arx             # Archive viewer/extractor
│   │   ├── executable_cln.tmpl        # Cleanup script (apt, brew, flatpak, docker, node)
│   │   ├── executable_dcm             # Single docker compose runner with retry-on-apparmor
│   │   ├── executable_dcms            # All docker-compose stacks under ~/servers/
│   │   ├── executable_goup            # Run command in current + parent directories
│   │   ├── executable_gpurr           # Git pull all repos
│   │   ├── executable_gpwiki          # Git push wiki
│   │   ├── executable_jot             # Encrypted S3 notes editor
│   │   ├── executable_mount-raid.tmpl # NFS mount helper
│   │   ├── executable_ollama-sync     # Update all ollama models
│   │   ├── executable_upd.tmpl        # System updater (apt, snap, flatpak, brew, bun)
│   │   ├── executable_update-dependencies   # Update project dependencies
│   │   ├── executable_git-*           # Git helper scripts
│   │   ├── executable_playbash        # Multi-host playbook runner (Node)
│   │   └── executable_playbash-{daily,weekly,hello,sample}  # playbash playbooks
│   ├── libs/
│   │   ├── bootstrap.sh               # Sources options.bash core modules
│   │   ├── playbash.sh                # Sidecar/event helpers sourced by playbash playbooks
│   │   └── maintenance.sh             # report_reboot/warn/action helpers + apt-history scanning
│   │                                  # (sourced by upd, cln; writes both colored output and
│   │                                  # JSON-lines events to $PLAYBASH_REPORT when set)
│   ├── private_share/                 # → ~/.local/share/ (private permissions)
│   │   ├── playbash/                  # playbash runner modules (render, inventory, sidecar)
│   │   ├── utils/                     # general Node helpers (comp, semver, nvm)
│   │   └── private_gnome-shell/       # GNOME shell extensions
│   └── vendors/
│       └── fzf-git.sh                 # fzf git integration
├── private_dot_ssh/                   # → ~/.ssh/ (SSH config)
├── run_onchange_before_install-packages.sh.tmpl  # Package installation script
└── run_once_after_install-vim.sh      # Vim setup
```

## Chezmoi naming conventions

Chezmoi uses special prefixes in source file names:

- **`dot_`** → `.` (e.g., `dot_bashrc` → `~/.bashrc`)
- **`private_`** → file/dir with restricted permissions (e.g., `private_dot_local` → `~/.local`)
- **`executable_`** → file is `chmod +x` (e.g., `executable_arx` → `~/.local/bin/arx`)
- **`.tmpl`** suffix → Go template processed with chezmoi data (OS, GUI flag, etc.)
- **`run_onchange_before_`** → script runs before apply when its content changes
- **`run_once_after_`** → script runs once after apply

Template data is defined in `.chezmoi.toml.tmpl`:
- `osId` — e.g., `linux-ubuntu`, `darwin`
- `osIdLike` — e.g., `linux-debian`, `darwin`
- `wsl` — true if running under WSL
- `codespaces` — true if running in GitHub Codespaces
- `hasGui` — whether the machine has a GUI

## Critical rules

- **Bash 4.0+** for all shell scripts. Start with `#!/usr/bin/env bash`.
- **`set -euCo pipefail`** and **`shopt -s expand_aliases`** at the top of every utility script.
- **Do not add comments or remove comments** unless explicitly asked.
- **Do not modify or delete test scripts** without understanding what they verify.
- All CLI utilities that use `options.bash` source it via `. ~/.local/libs/bootstrap.sh`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/uhop) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
