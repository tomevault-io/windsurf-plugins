---
trigger: always_on
description: This file provides guidance to coding agents working in this repository.
---

# AGENTS.md

This file provides guidance to coding agents working in this repository.

> IMPORTANT: Never consider backward compatibility. Eliminate legacy support by default.

## Critical Safety Rules (Read First)

These rules override all other instructions. Violations are unacceptable.

### Never Make Changes Irrecoverable

Absolutely forbidden:

- `git stash drop` or `git stash clear`
- `git reset --hard` without explicit user approval
- `git clean -fd` or similar destructive operations
- `rm -rf` on user files or directories
- Any command that permanently deletes user work

Required practices:

- Use `rip <path>` instead of `rm` for deletions (recoverable from graveyard)
- Use `git stash` when needed, but never drop stashes
- Allowed stash operations are `git stash`, `git stash list`, `git stash show`, and `git stash apply`
- `git stash pop` requires explicit user approval
- Preserve user changes; if uncertain, ask first
- Before any potentially destructive operation, stop and ask

If something is accidentally deleted:

1. Immediately attempt recovery (stash hash, reflog, `rip` graveyard, etc.)
2. Inform the user exactly what happened and what was recovered
3. Never hide or minimize deletion mistakes

## Repository Overview

This is a NixOS configuration using the Dendritic Pattern (organic configuration growth with automatic module discovery). Files can be moved and nested freely without breaking imports.

Canonical documentation lives under `docs/architecture/`.

## Nix Configuration

`flake.nix#nixConfig` carries only pre-evaluation settings needed before the
module graph is loaded:

- `abort-on-warn`
  - Value: `false`
  - Purpose: Don't abort on warnings
- `extra-experimental-features`
  - Value: `[ "pipe-operators" ]`
  - Purpose: Enable pipe operator syntax in Nix expressions
- `allow-import-from-derivation`
  - Value: `true`
  - Purpose: Required by IFD consumer `nix-doom-emacs-unstraightened`

Durable daemon and evaluator settings live in `modules/base/nix-settings.nix`.
Cache topology and download retry settings live in
`modules/hosts/common/nix-substituters.nix`. Inspect those owning files for
current values instead of duplicating the full `nix.settings` set here.

`build.sh` exports `NIX_CONFIG` only as a bootstrap overlay for the Nix commands
it launches before the target system configuration is active.

## Architecture and Module System

### Automatic Module Discovery

All Nix files are automatically imported as flake-parts modules. Files prefixed with `_` are ignored. Avoid literal path imports. Modules register under:

- `flake.nixosModules`
- `flake.homeManagerModules`

### Module Composition Pattern

Hosts compose modules from aggregator namespaces, not literal paths. Use `lib.hasAttrByPath` with `lib.getAttrFromPath` for optional modules to avoid ordering issues.

### Shared Host Modules (`modules/hosts/common/`)

Modules that apply to every host opted into the registry live under `modules/hosts/common/`. The registry is `flake.lib.nixos.hosts.<name>.shareCommon` (declared in `modules/hosts/common/registry.nix`).

Common modules contribute to the aggregate `flake.nixosModules.hosts-common` module. `modules/configurations/nixos.nix` imports that aggregate for each host whose registry entry has `shareCommon = true`, before importing the host-specific module so per-host overrides still win.

```nix
{ ... }:
let
  body = {
    networking.domain = "local";
  };
in
{
  flake.nixosModules.hosts-common.imports = [ body ];
}
```

Do NOT iterate over `flake.lib.nixos.hosts` with `lib.filterAttrs`/`lib.mapAttrs` from `modules/hosts/common/*.nix`. Host iteration belongs in `modules/configurations/nixos.nix`, which already owns NixOS system construction. Iterating from a common module that contributes to host configuration can trigger infinite recursion in the flake-parts module evaluator.

`modules/hosts/common/apps-enable.nix` carries the default-on baseline at `lib.mkOverride 1100`; per-host override files (e.g. `modules/tpnix/apps-enable.nix`) layer overrides at `lib.mkOverride 1000` so the host value wins. User overrides at default priority 100 still win over both. `modules/hosts/common/checks.nix` adds a flake-level `nix flake check` assertion that fails when a per-host override duplicates the common baseline value (silent no-op detection).

### Flake Input Deduplication

Use the generated README's "Flake Input Deduplication" section as the canonical
source for local flake input naming and follower relationships. Its source text
is `modules/readme.nix`.

### Repository Layout

- NixOS modules
  - Location: `modules/`
  - Notes: Auto-loaded. Per-host logic under `modules/system76` and `modules/tpnix`; cross-host shared logic under `modules/hosts/common`; other bundles grouped by domain.
- Shared derivations
  - Location: `packages/`
  - Notes: Common build logic shared between modules.
- Helper scripts
  - Location: `scripts/`
  - Notes: Operational tooling.
- Documentation
  - Location: `docs/`
  - Notes: Long-form references and local workflows. The NixOS manual mirror
    lives under `docs/nixos-manual/`.
- Secrets
  - Location: `secrets/`
  - Notes: Encrypted payloads managed via `sops.secrets`.
- Generated artifacts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Bad3r/nixos](https://github.com/Bad3r/nixos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
