---
trigger: always_on
description: A NixOS port of [basecamp/omarchy](https://github.com/basecamp/omarchy) (the
---

# omarchy-nix Agent Guide

A NixOS port of [basecamp/omarchy](https://github.com/basecamp/omarchy) (the
Quattro generation, [upstream PR #6231](https://github.com/basecamp/omarchy/pull/6231)).

> **START HERE:** read [`README.md`](README.md) and the docs in
> [`docs/`](docs/) before any work. Process-level checks (`ps aux`,
> `nix flake check`) are necessary but **not sufficient**: "verified" means
> behaviorally verified in a running desktop session.

## Hard rules

- **This project is vendored, not rewritten.** The upstream `basecamp/omarchy`
  tree (the `quattro` branch) is packaged as a Nix derivation; we do not
  hand-translate bash scripts, QML, or Lua into Nix. Updating to a newer
  Omarchy release means `nix flake lock --update-input omarchy-src` + rebuild,
  not editing modules.

## Architecture (one paragraph)

Upstream Quattro's "desktop" is a **single `quickshell` process** that provides
the bar, launcher, menus, notifications, OSDs, control panels, lock screen,
and polkit agent as plugins, plus a **Lua-based Hyprland config** (≥0.56),
~456 `omarchy-*` bash scripts in `bin/`, and a **TOML + template theme
engine**. Waybar/wofi/mako/hyprlock/hyprpaper/swaybg/polkit-gnome are gone
in Quattro. This port packages the upstream tree at `$out/share/omarchy`,
exports `OMARCHY_PATH`, and seeds `~/.config/hypr/hyprland.lua` so Hyprland's
autostart launches `quickshell -p $OMARCHY_PATH/shell`.

## Project layout

```
flake.nix              # inputs + outputs (packages, modules, checks, configs)
config.nix             # public omarchy.* option schema
pkgs/                  # derivations:
  omarchy.nix          #   vendoring: omarchy-src -> $out/share/omarchy
  omarchy-catalog.nix  #   Install/Remove menu catalog (nix-catalog.json)
  omarchy-migrations.nix  # migration classification manifest
  omarchy-runtime-manifest.nix  # Arch-mutator classification (fail-closed)
  omarchy-etc-manifest.nix      # upstream etc/ file classification
  migrations-nix/      #   NixOS adapter scripts for class "adapter"
  plymouth-omarchy-theme.nix  #   boot-splash theme
  sddm-omarchy-theme.nix      #   login theme + Hyprland greeter config
modules/lib/           #   option-value validators/serializers (Lua, env.d)
modules/nixos/         # NixOS module: env, runtime deps, Hyprland, themes
modules/home-manager/  # HM module: per-user config seed (hypr entry + stubs)
tests/desktop.nix      # automated desktop test (checks.omarchy-desktop)
tests/ux.nix           # behavioral acceptance (checks.omarchy-ux)
tests/fish.nix         # fish profile acceptance (checks.omarchy-fish)
skills/omarchy/        # NixOS-native agent skill (packaged + parity manifest)
example/               # demo consumer configuration.nix
docs/                  # install.md, options.md, UPSTREAM.md, vm.md,
                       # nix-best-practices.md; MAINTAINERS.md + SYSTEMS.md,
                       # decisions/ and superpowers/ are maintainer-internal
                       # (see the note below)
.forgejo/workflows/    # CI lanes: fast / vm / nightly (internal)
hosts/                 # maintainer host configs spliced into the flake (internal)
scripts/publish-public # sanitized snapshot publisher for the GitHub mirror (internal)
```

The Install/Remove menu is NixOS-native: catalog entries (browsers,
editors, terminals, AI agents, dev toolchains, services) resolve through
`omarchy-nix-add`/`omarchy-nix-remove` into `omarchy-packages.json`, which
`omarchy.managedPackagesFile` folds into the system at eval time; all the
NixOS-specific commands share one consumer-flake resolver
(`OMARCHY_NIX_FLAKE` → `~/omarchy-nix` → `~/Projects/omarchy-nix` →
`/etc/nixos`). `omarchy update` keeps the upstream UX but runs
`nix flake update` + `nixos-rebuild` against the consumer flake.

## Inputs

- `nixpkgs` → `nixos-26.05` (stable, so consumers on a stable NixOS install
  do NOT get shifted to unstable by `nixos-rebuild switch --flake`).
- `omarchy-src` → `github:basecamp/omarchy/quattro`, `flake = false` (it is not
  a flake; we vendor it). Upstream renamed the org to `omacom/omarchy`
  (2026-09); the old URL redirects, so the input is unchanged.
- `hyprland` → `github:hyprwm/Hyprland` (needs ≥0.56 for Lua config).
- `home-manager` → `github:nix-community/home-manager`, follows `nixpkgs`.
- `hermes-agent` → `github:NousResearch/hermes-agent` (the `hermes`
  default-agent CLI, consumed as its own flake — it brings its own nixpkgs).
- `quickshell`: this repo pins 0.3.1 (`pkgs/quickshell.nix`, injected as
  `omarchy.quickshellPackage`) because stable nixpkgs carries 0.3.0 — 0.3.1
  fixes the plugin-reload OSD side effect measured here and carries
  session-lock/wifi crash fixes. `checks.omarchy-desktop` verifies the shell
  loads and registers its instance, `checks.omarchy-quickshell-version`
  guards the pin, and `checks.omarchy-ux` asserts the OSD survives a plugin
  reload. Drop the pin (`pkgs/quickshell.nix`, the flake's packages entry,
  the wrapper injection and the version check) when the nixpkgs pin carries
  0.3.1 or newer.

## Verification (every stage)

1. `nix flake check` must pass before any commit. This includes
   `checks.omarchy-desktop`, an automated NixOS test that boots a VM with

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zicochaos/omarchy-nix](https://github.com/zicochaos/omarchy-nix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
