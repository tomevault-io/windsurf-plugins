---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single Nix flake defining a complete NixOS distribution ("Oligarchy") targeting the **Framework 16 AMD 7040**. Configuration-as-code: six `nixosConfigurations` (`nixos` is the primary; `nixos-asher` is the maintainer's real Framework 16 — `nixos` extended with the committed `hosts/asher/` layer, see Flake composition below; `nixos-fw13`, `nixos-intel`, `nixos-optimus` are the alternate laptops; `builder` is the headless CI box), an installer ISO, nineteen locally-vendored `path:` sub-flakes (several containing Rust programs), and a Home Manager user config. The build artifact is an OS — there is nothing to "run".

<!-- truth:claim
id: source-of-truth
kind: file_exists
path: docs/architecture.md
-->
The README is heavy in-character satire ("war machine", fake legal decrees). Ignore the tone; the technical tables near the bottom, `flake.nix`, and `docs/architecture.md` are the source of truth.

Large: tens of thousands of lines of Nix, Rust and Shell. `docs/architecture.md` §1a carries the per-language breakdown; treat its exact figures as a snapshot, not a contract, because nothing checks them and they were already stale once. The useful part is which subsystems are dense versus merely large: `home/` is broad and mostly not load-bearing, while `oligarchy-p2p`, `oligarchy-plugins`, `warroom` and `reliquary` are where the invariants live.
<!-- truth:end --> Two things to expect when reading: **19% of the Nix and Rust is whole-line comments** (the reason a decision was made lives next to it, not only in a roadmap), and **14% of the Rust is `#[cfg(test)]`** — which still undercounts testing, because the expensive assertions are VM gates rather than unit tests.

## Where the docs are

`docs/` is the long form. This file states the rules; these state why, and are the place to read before a non-trivial change to the subsystem they cover.

| doc | covers |
|---|---|
| `architecture.md` | straight technical reference to the whole tree; §1a is the scale/density breakdown |
| `plugins-roadmap.md` | `oligarchy-plugins` design, staging record, per-stage known gaps. Superset of the plugin landmines below |
| `p2p-substituter-roadmap.md` | `oligarchy-p2p` design, trust argument, staging, known gaps. Superset of the p2p landmines below |
| `mcp-servers-roadmap.md` | the ten read-only MCP aspects; adding one |
| `oligarchy-forge-roadmap.md`, `oligarchy-forge-research.md` | forge design + roadmap (Phases 0-1 built), and the prior art it rests on |
| `security-hardening.md` | the hardening/egress/malware-shield posture |
| `vpn-windscribe.md` | `custom.vpn` — Windscribe over WireGuard, the sops slot, and why the tunnel is on demand with no kill switch |
| `dgpu-steam-forcing.md` | dGPU-vs-iGPU client rendering; why Hyprland's backend never moves |
| `secure-boot-enrollment.md`, `bios-uma-unlock.md` | firmware procedures — read before running either |
| `dcf-mesh-agent.md` | the read-write UDP mesh endpoint kept out of the MCP surface |
| `exsecutor-kernel-roadmap.md` | the Exsecutor/ring-0 agent-sandbox assessment: what the hardware actually allows, the six language prerequisites before any kernel object, and which phases are already built here. Claims about this tree are TrvthNvke-bound; Exsecutor claims are pinned by commit |
| `localization-roadmap.md` | i18n/l10n design: `custom.locale.*` contract, catalogs, installer round-trip (`oligarchy-adopt`), gates. Stages 0-2 landed (`custom.locale`, `oligarchy-adopt`); §3 is the measured part; §6a records the installer reversal |
| `installer.md` | what the ISO installs (an `installTargets` entry via `mkInstalled`, not plain NixOS), how answers map to `custom.locale.*`/`custom.user.*`, `oligarchy-install`, the ISO's SSH. `installer/README.md`: the vendored-from-ArchibaldOS rule |

<!-- truth:claim
id: subsystem-readmes
kind: dir_exists
path: modules/oligarchy-p2p/
-->
Subsystem READMEs carry the same role one level down: `modules/oligarchy-p2p/`, `modules/oligarchy-plugins/`, `modules/mcp-servers/`, `modules/demod-talk/`, `modules/minecraft/`, `modules/hypr-controller/`.
<!-- truth:end -->

## Common commands

All commands run from the repo root.

```bash truth:ignore
# Enter the dev shell (Nix LSP `nil`, nixpkgs-fmt, qemu, virt-manager, network/debug tools)
nix develop

# Format Nix files. Use nixpkgs-fmt, NOT `nix fmt`: the flake's `formatter` is
# nixfmt-rfc-style, a different style that will reformat whatever it touches.
nixpkgs-fmt <file.nix>

# Evaluate/build the full system without switching (catches most errors).
# .#nixos is the fresh-clone-minimal config — pure eval, so it never sees a
# ~/.config/oligarchy/{local,state}.nix override. .#nixos-asher is the
# maintainer's real Framework 16, and is just as pure: its personal toggles
# are the committed hosts/asher/ layer, not an out-of-repo file.
nix build .#nixosConfigurations.nixos.config.system.build.toplevel
nix build .#nixosConfigurations.nixos-asher.config.system.build.toplevel

# Apply the config to a running NixOS host. asher's machine needs no flags —
# .#nixos-asher is .#nixos extended with the committed hosts/asher/ layer

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ALH477/Oligarchy](https://github.com/ALH477/Oligarchy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
