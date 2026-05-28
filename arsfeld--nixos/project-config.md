---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a personal NixOS configuration repository that manages multiple machines using Nix Flakes and flake-parts. It includes configurations for servers (galactica, basestar), embedded devices (R2S, Raspberry Pi), and desktop systems (raider, blackbird).

## Key Commands

### Development Environment
```bash
nix develop                    # Enter dev shell (required for most operations)
just fmt                       # Format all Nix files with alejandra
just build <hostname>          # Build a host config locally
```

### Deployment (via Colmena, default)
```bash
just deploy galactica           # Deploy to one host
just deploy galactica basestar  # Deploy to multiple hosts in parallel
just boot galactica             # Boot activation (next reboot)
just test galactica             # Test without activating
just dry-run galactica          # Show changes without downloading/building
just reboot galactica           # Deploy and reboot (kernel changes)
just info                      # List all known hosts
```

nixos-rebuild fallback: `just nr-deploy <host>`, `just nr-boot <host>`, `just nr-test <host>`

deploy-rs is available but currently broken with Nix 2.32+ (`just deploy-rs`, `just boot-rs`).

All hosts are reached via Tailscale: `<hostname>.bat-boa.ts.net`.

### Testing Changes
```bash
nix build .#nixosConfigurations.<hostname>.config.system.build.toplevel
```

### Secret Management

```bash
nix develop -c sops secrets/sops/<hostname>.yaml    # Create/edit host secrets
nix develop -c sops --decrypt secrets/sops/basestar.yaml  # View decrypted
nix develop -c sops updatekeys secrets/sops/<file>.yaml  # Re-encrypt after key changes
```

Configured via `.sops.yaml`. All hosts use `constellation.sops.enable = true`. Use standard `sops.secrets` options. Common/shared secrets: `config.constellation.sops.commonSopsFile`.

### Available Hosts
- **galactica** - Main server: media services, databases, backups. Hosts internal services on `*.arsfeld.one` via cloudflared tunnel (wildcard ingress)
- **basestar** - Public-facing server (BSG Cylon Basestar): hosts services on `*.arsfeld.dev` (blog, plausible, planka, siyuan)
- **raider** - Desktop workstation: GNOME, gaming, development
- **router** - Custom network device (no constellation modules, standalone config)
- **r2s** - ARM-based router (NanoPi R2S)
- **raspi3** - Raspberry Pi 3
- **blackbird** - ASUS ROG Zephyrus G14 laptop (BSG Blackbird — custom stealth ship)
- **pegasus** - Secondary server (BSG Battlestar Pegasus)
- **octopi** - OctoPrint device

For hardware specs (CPU, RAM, disks), see [HARDWARE.md](HARDWARE.md).

## Architecture Overview

### Flake Structure

The flake uses **flake-parts** to organize outputs into modules under `flake-modules/`:
- **`lib.nix`** - Core utilities: `mkLinuxSystem`, overlays, `baseModules`, `homeManagerModules`. Uses **haumea** to recursively auto-load all files from `modules/` and `packages/` directories.
- **`hosts.nix`** - Auto-discovers hosts by scanning `hosts/` for directories with `configuration.nix`. Automatically includes `disko-config.nix` if present.
- **`deploy.nix`** - deploy-rs configuration for each host
- **`colmena.nix`** - Colmena deployment with cross-compilation support for aarch64
- **`dev.nix`** - Development shell, formatter, git hooks, custom packages
- **`checks.nix`** - Flake checks (router NixOS test)
- **`images.nix`** - System image generators (SD cards, kexec)

### Module Auto-Discovery

All `.nix` files under `modules/` are loaded automatically by haumea - no explicit imports needed. To add a new module, create a file in `modules/` (or a subdirectory) and it will be available to all hosts. Hosts then selectively enable modules via `constellation.<module>.enable = true`.

### Constellation Modules (`modules/constellation/`)

Opt-in feature modules that hosts compose. Key modules:

| Module | Purpose |
|--------|---------|
| `common.nix` | Base config: Nix flakes, caches, SSH, Tailscale, Avahi |
| `users.nix` | User accounts, SSH keys, sudo |
| `sops.nix` | sops-nix infrastructure (age keys, default paths) |
| `services.nix` | **Central service registry**: ports, auth, CORS, Tailscale exposure |
| `media.nix` | **Container orchestration**: Plex, *arr, Stash, Nextcloud, etc. |
| `podman.nix` / `docker.nix` | Container runtimes |
| `backup.nix` | Automated rustic/restic backups |
| `vpn-exit-nodes.nix` | Tailscale exit nodes via AirVPN/Gluetun |
| `gnome.nix` / `cosmic.nix` / `niri.nix` | Desktop environments |
| `development.nix` | Dev tools (Docker, Node, Python, Go, Rust) |
| `gaming.nix` | Gaming environment |
| `metrics-client.nix` / `logs-client.nix` | Observability agents |
| `observability-hub.nix` | Central Prometheus/Loki hub |
| `home-assistant.nix` | Home automation |
| `virtualization.nix` / `project-vms.nix` | KVM/libvirt VMs |

### Media Configuration Variables (`modules/media/config.nix`)

Shared variables consumed by media services via `config.media.config`:
- `configDir` = `/var/data` - Service config/data directory

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arsfeld/nixos](https://github.com/arsfeld/nixos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
