---
trigger: always_on
description: This repository manages a multi-PC NixOS lab using Nix Flakes,
---

# AGENTS.md

This repository manages a multi-PC NixOS lab using Nix Flakes,
Disko, and Colmena. A controller PC deploys to student workstations
over a LAN-only network (no internet on clients).

All lab-specific settings (user names, PC count, network layout, passwords,
locale, etc.) are parameterized in `lab-config.nix` and imported by `flake.nix`.

## Project Structure

```
flake.nix                  # Entry point: imports lab-config.nix, host generation + netboot + Colmena
flake.lock                 # Pinned inputs (nixpkgs nixos-25.11, disko)
LICENSE                    # MIT license
lab-config.nix             # Lab configuration (edit for your environment)
disko-uefi.nix             # NixOS wrapper for the shared Disko layout
lib/
  disko-layout.nix         # Shared Disko layout function (device + student user)
setup.sh                   # Installer script for PXE-booted client PCs
pkgs/
  veyon.nix                # Veyon package derivation (not in nixpkgs)
  gnome-remote-desktop.nix # gnome-remote-desktop overlay (VNC + multi-session)
modules/
  common.nix               # Shared system config (GNOME, packages, shells, locale, services)
  hardware.nix             # Generic hardware detection (replaces per-host hardware-configuration.nix)
  networking.nix           # Hostname + static IP with shared iface name
  users.nix                # User accounts (admin + teacher + student, veyon-master group)
  cache.nix                # Binary cache client (points to controller's Harmonia)
  filesystems.nix          # Btrfs subvolume mount declarations
  home-reset.nix           # Student home directory templating + boot-time reset
  veyon.nix                # Veyon service, public key, firewall, base config
scripts/
  install-controller.sh    # Live USB bootstrap installer for controller with disk selection
  run-harmonia.sh          # Launches Harmonia binary cache server
  run-pxe-proxy.sh         # ProxyDHCP + TFTP + HTTP netboot server (external DHCP compatible)
  lib/lab-meta.sh          # Shared helper: loads labMeta from the flake for shell scripts
  create-home-template.sh  # Builds clean home directory template
  home-reset.sh            # Boot-time snapshot rotation + home reset
  cmd-screensaver.sh       # TTE screensaver animation loop
  launch-screensaver.sh    # Fullscreen Ghostty screensaver launcher
  screensaver-monitor.sh   # GNOME idle watcher for screensaver
assets/
  backgrounds/             # Ristretto wallpapers (random at each home-reset)
  logo.txt                 # ASCII art for screensaver
  mimeapps.list            # Default browser = Chromium
  vscode-settings.json     # VS Code defaults
```

Generated locally during setup and committed in the lab repo:
- `public-key`
- `id_ed25519.pub`
- `veyon-public-key.pem`

## Build / Deploy Commands

```sh
# Evaluate a single host config (syntax/type check without building)
nix eval .#nixosConfigurations.pc01.config.system.build.toplevel --no-write-lock-file

# Build a single host (full build, outputs to ./result)
nix build .#nixosConfigurations.pc01.config.system.build.toplevel

# Build all client closures
nix build .#nixosConfigurations.pc{01..20}.config.system.build.toplevel

# Rebuild and activate on the local machine (controller)
sudo nixos-rebuild switch --flake .#pcNN --no-write-lock-file

# Deploy to all lab PCs via Colmena
colmena apply --on @lab

# Deploy to a single PC
colmena apply --on pc05

# Build netboot artifacts
nix build .#nixosConfigurations.netboot.config.system.build.kernel --out-link result-kernel
nix build .#nixosConfigurations.netboot.config.system.build.netbootRamdisk --out-link result-initrd
nix build .#nixosConfigurations.netboot.config.system.build.netbootIpxeScript --out-link result-ipxe
```

There are **no tests, linters, or formatters** configured in this repository.
To validate changes, build the affected host configuration (`nix build`).

## Architecture Notes

- Hosts pc01-pcNN are generated programmatically via `builtins.genList` + `mkHost`/`mkColmenaHost` in `flake.nix`, with the controller defined separately.
- Hostname + static IP are centralized in `flake.nix` (derived from `networkBase` + host number) and applied in `modules/networking.nix`. Each PC gets both a DHCP address and a static address on the same interface.
- The controller has two relevant IPs: `masterIp` (static, `networkBase.masterHostNumber`) used by Colmena and the binary cache for day-to-day deploys, and `masterDhcpIp` (dynamic, assigned by the institutional DHCP server) used only during PXE/netboot client installation. If the DHCP lease changes, `masterDhcpIp` in `lab-config.nix` must be updated and netboot artifacts rebuilt before the next PXE session.
- Custom settings flow from `flake.nix` via `specialArgs` (`labSettings`, `hostName`, `hostIp`) to modules that need them.
- `labSettings` is a plain attribute set containing all configurable values: user names (`teacherUser`, `studentUser`), passwords, SSH key, network settings, locale/timezone, homepage URL, git identity, and more.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giovantenne/nixos-lab](https://github.com/giovantenne/nixos-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
