---
trigger: always_on
description: NixOS configuration for `thixos`, a WSL2-based development machine.
---

# .dotfiles

NixOS configuration for `thixos`, a WSL2-based development machine.

## Repository Structure

```
.dotfiles/
├── flake.nix              # Top-level flake: NixOS config + inputs
├── flake.lock
├── nixos/
│   └── configuration.nix  # System config (~700 lines, the core of everything)
├── config/
│   ├── zellij/            # Zellij config, layouts, plugin config
│   ├── starship.toml      # Shell prompt
│   └── wezterm.lua        # Terminal emulator (runs on Windows, copied via symlink)
├── bespoke/
│   └── zellij/
│       └── dim-unfocused/ # Custom Zellij build + shader plugin (has its own AGENTS.md)
├── templates/             # Nix flake templates
└── .zshrc, .gitconfig, etc.
```

## How It Builds

The top-level `flake.nix` produces a single NixOS configuration (`thixos`):

- **Inputs**: `nixpkgs` (unstable), `nixos-wsl`, `llm-agents` (Claude Code, Gemini, Codex), `dim-unfocused` (local path flake for custom Zellij)
- **Output**: `nixosConfigurations.thixos`

The `dim-unfocused` sub-flake (at `bespoke/zellij/dim-unfocused/`) has its own inputs including a Zellij fork. It produces a custom Zellij binary and a WASM plugin. See `bespoke/zellij/dim-unfocused/AGENTS.md` for deep details on the shader system.

### Deploying changes

```bash
# From the repo root:
nswitch    # alias defined in configuration.nix — runs nixos-rebuild switch
```

## configuration.nix

This is the heart of the system. It defines:

### Custom shell scripts (~half the file)

The configuration embeds several substantial shell applications as `writeShellApplication` derivations:

- **`session-picker`** — Zellij session manager. Lists existing sessions, creates new ones, clones GitHub repos. Launched on shell login.
- **`command-palette`** — Context-aware fzf command launcher with ~25 commands. Invoked via Zellij keybinding.
- **`code-session`**, **`claude-session`**, **`gemini-session`** — Launch dev/LLM sessions in Zellij.
- **`gcloud-switch`**, **`gcloud-reauth`** — GCP project/cluster switching.
- **`zed`** — Interop wrapper to launch Windows Zed editor from WSL.
- **`open-browser`** — Opens URLs in Windows Chrome from WSL.

### Shell environment (zsh)

- Starship prompt
- Auto-attaches to Zellij on login
- Zoxide + fzf for directory navigation
- Dynamic pane titles (`repo:branch` format via precmd/preexec hooks)
- Environment: `ZELLIJ_CONFIG_DIR`, `BROWSER`, `EDITOR` (micro), `MANPAGER` (bat)

### System packages

Dev tools (zellij, git, gh, ripgrep, fd, bat, eza, delta, lazygit, difftastic), cloud tools (gcloud, kubectl), LLM agents (claude-code, gemini-cli, codex).

### Config file management

Files from `config/` are symlinked into `/etc/` or `~/.config/` via `environment.etc`. The WezTerm config is copied to the Windows filesystem.

## Working with this repo

- **Zellij is custom-built** from a fork with pane shader support. Don't update the `dim-unfocused` flake input without checking the fork is in sync. See `bespoke/zellij/dim-unfocused/AGENTS.md`.
- **Zellij rebuild takes ~20 minutes** (Rust). Avoid changes to the fork unless necessary. The plugin and shader WASM rebuild in seconds.
- **The `config/zellij/config.kdl`** loads the dim-unfocused plugin. If the plugin WASM path changes, update it there.
- **Shell scripts in `configuration.nix`** are the most frequently edited part. They rebuild instantly with `nswitch`.
- **Flake lock updates**: The top-level lock pins transitive inputs from sub-flakes. After updating `bespoke/zellij/dim-unfocused/flake.lock`, also run `nix flake update` at the top level to propagate.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tteggel)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tteggel)
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
