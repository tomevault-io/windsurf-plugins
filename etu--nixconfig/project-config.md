---
trigger: always_on
description: > **Important:** Whenever you make changes to this repository, update this file
---

# AGENTS.md — Guide for AI Agents

> **Important:** Whenever you make changes to this repository, update this file
> **and** [README.md](./README.md) before finishing. This applies to: adding or
> removing hosts, adding Justfile recipes, adding/changing secrets, changing
> modules or CI workflows, and discovering new upgrade breakage patterns.
> A reminder checklist is at the **bottom of this file**.

---

## Repository Overview

This is a personal NixOS configuration repository for **etu** (Elis Hirwing).
It manages multiple machines using [Nix Flakes](https://nixos.wiki/wiki/Flakes),
[home-manager](https://github.com/nix-community/home-manager),
[deploy-rs](https://github.com/serokell/deploy-rs) for remote deployments, and
[agenix](https://github.com/ryantm/agenix) for secret management.

The flake is structured using [blueprint](https://github.com/numtide/blueprint)
which provides a conventional layout that is recursively merged with custom
flake outputs.

---

## Top-Level Files and Directories

| Path | Description |
|------|-------------|
| `flake.nix` | Main flake definition: inputs, blueprint wiring, deploy-rs nodes, and the live-iso package alias |
| `flake.lock` | Locked versions of all flake inputs |
| `pubkeys.nix` | All SSH public keys for users and host systems — standalone, no imports |
| `secrets-registry.nix` | **Single source of truth** for all agenix secrets: file path, owner/path/symlink options, and the list of host keys allowed to decrypt each secret |
| `secrets.nix` | Auto-derived from `secrets-registry.nix` — do not edit directly; consumed by the agenix CLI |
| `secrets/` | Directory of agenix-encrypted secret files (`.age`) |
| `hosts/` | Per-machine NixOS configurations |
| `modules/` | Reusable NixOS (`modules/nixos/`) and home-manager (`modules/home/`) modules |
| `packages/` | Custom Nix packages defined in this repo |
| `devshell.nix` | `nix develop` shell with all tools needed for day-to-day work |
| `Justfile` | Task runner (via [just](https://just.systems/)) wrapping common nix, build, deploy, format, and update commands |
| `.envrc` | [direnv](https://direnv.net/) integration — runs `use flake` automatically |
| `.statix.toml` | Configuration for [statix](https://github.com/nerdypepper/statix) (disables `repeated_keys` check) |
| `.yamllint` | YAML lint configuration |
| `.github/workflows/` | CI/CD GitHub Actions workflows |

---

## Hosts

Each host lives in `hosts/<hostname>/` and is automatically picked up by blueprint.

| Host | Type | Deployment method | Notes |
|------|------|-------------------|-------|
| `desktop-elis` | Desktop | `nixos-rebuild` locally | Primary desktop for etu |
| `desktop-caroline` | Desktop | `nixos-rebuild` locally | Desktop for Caroline |
| `laptop-private-elis` | Laptop | `nixos-rebuild` locally | Private laptop (T495); pushes ZFS snapshots to `server-main-elis` |
| `laptop-private-caroline` | Laptop | `nixos-rebuild` locally | Private laptop for Caroline |
| `laptop-work-elis` | Laptop | `nixos-rebuild` locally | Work laptop; pushes ZFS snapshots to `server-main-elis` |
| `server-main-elis` | Server | `deploy .#server-main-elis` | Home file server; also a Nix build machine, ZFS snapshot target, runs Home Assistant |
| `server-sparv` | Server | `deploy .#server-sparv` | On-location server for speliarvika.se, LAN cache, game servers (Valheim, Project Zomboid, Minecraft) |
| `vps06` | VPS | `deploy .#vps06` | Runs Gitea, ip.failar.nu, Matrix homeserver |
| `live-iso` | ISO | `nix build .#iso` | Live ISO of this config; not deployed remotely |

---

## Modules

### `modules/nixos/`

Reusable NixOS modules covering:

| Directory | Purpose |
|-----------|---------|
| `base/` | Base system settings shared across hosts |
| `data/` | Defines `config.etu.data` option: derives `ageModules` from `secrets-registry.nix` and imports `pubkeys.nix` |
| `development/` | Development tooling options |
| `games/` | Gaming-related configuration |
| `graphical/` | Graphical desktop (Sway/Wayland) stack |
| `services/` | Various service configurations (e.g. Nextcloud, Gitea, Home Assistant) |
| `theme/` | System-wide theming (Catppuccin) |
| `user/` | User account definitions |
| `work/` | Work-specific settings |
| `default.nix` | Module entry point, imports all sub-modules |

### `modules/home/`

Home-manager modules for user-level configuration:

| Directory | Purpose |
|-----------|---------|
| `alacritty/` | Alacritty terminal emulator config |
| `emacs/` | Emacs configuration |
| `firefox/` | Firefox browser settings |
| `fish/` | Fish shell configuration |
| `flatpak-overrides/` | Flatpak permission overrides |
| `foot/` | Foot terminal emulator config |
| `graphical-dotfiles/` | Miscellaneous graphical dotfiles |
| `htop/` | htop config |
| `kanshi/` | kanshi (display management) config |
| `mako/` | mako notification daemon config |
| `sway/` | Sway compositor config |
| `tmux/` | tmux config |
| `vscode/` | VS Code settings and extensions |
| `waybar/` | Waybar (status bar) config |

---

## Development Environment

Enter the dev shell with:

```sh
nix develop
# or, with direnv configured:
direnv allow
```

The shell (defined in `devshell.nix`) provides:

- `just` — task runner

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [etu/nixconfig](https://github.com/etu/nixconfig) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
