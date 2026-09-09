---
trigger: always_on
description: Foundry is my public infra monorepo: the NixOS/home-manager config, plus:
---

# Foundry Conventions for LLMs

## Repository

Foundry is my public infra monorepo: the NixOS/home-manager config, plus:

- `projects/website/` — the site served at m7.rs, vendored from
  `github:misterio77/website` with full history and wired as a local flake input
  (`path:./projects/website`), so the monorepo builds it from its own tree; still
  mirrored to the standalone repo.
- `pkgs/wallpapers/` — my wallpaper collection. Each entry in `list.json` carries a
  committed `sourceColor` (matugen's extracted seed), and they're exposed as
  `pkgs.wallpapers` plus a Hydra-cached `wallpapers` package (imgur is upstream).
- `lib/` — a from-scratch pure-Nix implementation of Material You color science
  (`math`, `hct`, `palettes`, `scheme`, `material-you`), validated bit-for-bit
  against matugen. `modules/home-manager/colors.nix` uses it via
  `outputs.lib.material-you.generateColorscheme` to produce colorschemes during
  evaluation — no matugen, no import-from-derivation. Adding a wallpaper needs a
  one-time `matugen image` run to seed its `sourceColor` in `list.json`.

## Commit Messages

Conventional commits: `type(scope): description`

- `type`: `feat`, `fix`, `refactor`, `chore`, `WIP`
- `scope`: path-based, reflecting what part of the config changed. Examples:
  - `home/{feature}` for home-manager features: `home/calendar`, `home/opencode`, `home/helix`
  - `{host}` or `{host}/{service}` for host-specific: `pleione`, `alcyone/firefly`, `merope/recyclarr`
  - Just the component for shared/global: `grafana`, `minecraft`, `recyclarr`
  - `projects/{name}` for vendored project sources: `projects/website`
  - `lib` / `wallpapers` for the color engine and wallpaper collection
- Message is lowercase, no period at end.

### Flake Lock Bumps

When describing a `flake.lock` bump (e.g. after `nix flake update <input>`), summarize
what actually changed in the bumped input(s), not just the revision hashes:

1. Get the old → new revisions from the `nix flake update` output or `flake.lock` diff.
2. Fetch the upstream changelog between them (`gh`/GitHub compare API, or a local clone
   in `/tmp`): `curl -s https://api.github.com/repos/<owner>/<repo>/compare/<old>...<new>`.
3. In the commit body, note the short hash range and a brief bullet list of the
   meaningful changes (commits/files), so the diff is reviewable without leaving the repo.

Example:

```
chore(flake): update website input

Update website from f6c09b0b to 70386bb7 (2 commits, docs-only):
- add _src/llms.txt
- remove stale _src/portfolio.md and its references in llms.txt
```

## Directory Structure

```
.
├── home/gabriel/          # Home Manager user config
│   ├── features/          #   Feature modules (cli/, desktop/, productivity/, helix/, etc.)
│   │   └── {feature}/
│   │       ├── default.nix  # Feature flag + imports
│   │       └── *.nix        # Specific tool configs
│   ├── global/            #   Always-imported config (xdg, etc.)
│   ├── {hostname}.nix     #   Per-host home-manager config
│   └── generic.nix        #   Non-impermanence fallback
├── hosts/
│   ├── nixos/             # NixOS host configs
│   │   ├── common/        #   Shared across hosts
│   │   │   ├── global/    #     Always-imported
│   │   │   ├── optional/  #     Opt-in modules
│   │   │   └── secrets.yaml #   SOPS-encrypted shared secrets
│   │   └── {hostname}/    #   Per-host (atlas, maia, alcyone, celaeno, merope, pleione, taygeta)
│   │       ├── default.nix #    NixOS module
│   │       ├── hardware-configuration.nix
│   │       └── secrets.yaml #   Host-specific secrets (optional)
│   └── system-manager/    # Non-NixOS System Manager host configs
├── modules/               # Custom NixOS & HM modules
│   ├── nixos/
│   └── home-manager/
├── overlays/              # Package overlays and patches (incl. pkgs.wallpapers)
│   └── default.nix
├── pkgs/                  # Custom packages
│   ├── wallpapers/        #   Wallpaper collection + committed source colors
│   └── default.nix
├── lib/                   # Pure-Nix Material You color engine
│   ├── math.nix           #   hand-rolled float transcendentals
│   ├── hct.nix            #   sRGB<->CAM16<->HCT + solver
│   ├── palettes.nix       #   tonal palettes
│   ├── scheme.nix         #   DynamicColor MD3 roles
│   └── material-you.nix   #   generateColorscheme (used by colors.nix)
├── projects/              # Vendored public projects
│   └── website/           #   github.com/misterio77/website (served at m7.rs)
├── flake.nix              # Flake entry point
├── deploy.sh              # nixos-rebuild wrapper
└── .sops.yaml             # SOPS encryption keys
```

## Code Style

- **Formatter**: Alejandra (`nix fmt <file>`). ALWAYS format after edits. Never format unmodified files.
- **Indentation**: 2 spaces, no tabs.
- **Line endings**: LF, final newline, trimmed trailing whitespace.
- **Nix conventions**:
  - Top-level modules are functions taking `{pkgs, lib, config, inputs, ...}`.
  - Use `lib` from `nixpkgs.lib // home-manager.lib` (merged, already in `outputs.lib`).
  - Feature-flag modules use a `default.nix` with a boolean `enable` option gating imports.
  - Prefer `lib.mkOption` / `lib.mkEnableOption` for new options.

## Secrets


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Misterio77/Foundry](https://github.com/Misterio77/Foundry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
