---
trigger: always_on
description: This repository is a Nix flake-based dotfiles setup for macOS using:
---

# AGENTS.md (repo guidance for coding agents)

This repository is a Nix flake-based dotfiles setup for macOS using:
- Home Manager for user configuration
- nix-darwin for system configuration
- Neovim config managed via Home Manager (Lua configs stored in-repo)

This file is written for agentic coding tools working in this repo.

## Repository entrypoints

- Flake entrypoint: `flake.nix`
- Home Manager host: `homeConfigurations.srt-l02-sekhmet`
- nix-darwin host: `darwinConfigurations.srt-l02-sekhmet`
- NixOS home server: `nixosConfigurations.srt-n01-rivendell`

Key module roots:
- Home Manager modules: `modules/*` (mix of `default.nix` modules and single-file modules)
- macOS (nix-darwin) modules: `modules/mac/*.nix`
- NixOS host modules: `hosts/nixos/*`
- Neovim Lua configs: `modules/nvim/lua/*.lua`

Notes:
- `.secrets` is expected to be git-crypt’d (see `Readme.md`). Avoid editing/committing secrets.
- A `result` path may exist as a Nix build output/symlink; it may be broken/missing.
- Avoid committing machine-local artifacts (e.g. `.DS_Store`).

---

## Build / Apply commands

### Apply user (Home Manager)
Preferred:
- `home-manager switch --flake "./#srt-l02-sekhmet" -b backup`

Readme-compatible:
- `nix run --no-write-lock-file --inputs-from . home-manager#home-manager -- switch --flake "./#srt-l02-sekhmet"`

Script equivalent:
- `./modules/system-management/apply-user.sh`

### Apply system (nix-darwin)
Preferred:
- `sudo darwin-rebuild switch --flake ~/.dotfiles/.#srt-l02-sekhmet`

Script equivalent:
- `./modules/system-management/apply-system-mac.sh`

### Apply home server (NixOS)
Host:
- Hostname: `srt-n01-rivendell`
- Flake output: `nixosConfigurations.srt-n01-rivendell`
- Host config root: `hosts/nixos/srt-n01-rivendell/`
- Normal remote access: SSH over Tailscale, `ssh nixypanda@srt-n01-rivendell`
- Current known Tailscale IP: `100.127.3.54`
- Current known LAN IP: `192.168.1.76`

Preferred remote switch from the Mac:
- `env NIX_SSHOPTS='-i /Users/nixypanda/.ssh/github-key -o StrictHostKeyChecking=accept-new' nix run nixpkgs#nixos-rebuild -- switch --flake .#srt-n01-rivendell --build-host nixypanda@100.127.3.54 --target-host nixypanda@100.127.3.54 --no-reexec --use-substitutes --ask-sudo-password`

Build only on the server over SSH:
- `env NIX_SSHOPTS='-i /Users/nixypanda/.ssh/github-key -o StrictHostKeyChecking=accept-new' nix run nixpkgs#nixos-rebuild -- build --flake .#srt-n01-rivendell --build-host nixypanda@100.127.3.54 --target-host nixypanda@100.127.3.54 --no-reexec --use-substitutes`

Local on-server equivalent:
- `sudo nixos-rebuild switch --flake ~/.dotfiles#srt-n01-rivendell`

Server notes:
- Tailscale is enabled declaratively on both the Mac and NixOS server.
- The server keeps XFCE/LightDM enabled for local monitor/keyboard recovery.
- The `nixypanda` login shell on the server should remain Bash. Nushell is installed and configured, but making it the login shell breaks ordinary SSH remote commands such as `ssh host 'echo ---; hostname'`.
- `usbutils` and `usb-modeswitch` are Linux-only packages in `modules/cli.nix`; keep them guarded with `lib.optionals stdenv.hostPlatform.isLinux`.

### Update flake lockfile
Preferred:
- `nix flake update --flake .`

Script equivalent:
- `./modules/system-management/update-dots.sh`

When removing a flake input manually:
- Remove it from `inputs` in `flake.nix`.
- Remove it from the `outputs` argument pattern and any overlays/modules that reference it.
- Remove the node from `flake.lock`.
- Remove the corresponding entry from `nodes.root.inputs` in `flake.lock`.
- Verify with `rg -n -i "<name>|<name variants>" .`.

### “What will build?” forecast (optional)
- `./modules/system-management/build-forecast-user.sh`

This uses `nix-forecast` and writes full output to `/tmp/nix-forecast.txt`.

### Dangerous cleanup script (do not run unless asked)
- `./modules/system-management/clean-system.sh`

This prunes docker volumes/images, deletes caches, and runs Nix garbage collection.

---

## Tests / Validation (the “single test” equivalents)

This repo doesn’t have conventional unit tests.
Treat “tests” as:
- Nix evaluation/build validation
- Lint/format checks (Nix/Lua/shell/markdown)

### Fastest: evaluate only (preferred for small changes)
Home Manager activation derivation path:
- `nix eval --raw .#homeConfigurations.srt-l02-sekhmet.activationPackage.drvPath`

nix-darwin system derivation path:
- `nix eval --raw .#darwinConfigurations.srt-l02-sekhmet.system.drvPath`

NixOS server system derivation path:
- `nix eval --raw .#nixosConfigurations.srt-n01-rivendell.config.system.build.toplevel.drvPath`

If evaluation fails, re-run with:
- `--show-trace`

### Build only what you need
Build Home Manager activation package:
- `nix build .#homeConfigurations.srt-l02-sekhmet.activationPackage`

Build nix-darwin system derivation:
- `nix build .#darwinConfigurations.srt-l02-sekhmet.system`

Build NixOS server system derivation:
- On the server or with a Linux remote builder:
  `nix build .#nixosConfigurations.srt-n01-rivendell.config.system.build.toplevel`
- From the Mac, prefer `nixos-rebuild --build-host nixypanda@100.127.3.54`
  because local `x86_64-darwin` cannot build Linux-only derivations.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nixypanda/dotfiles](https://github.com/nixypanda/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
