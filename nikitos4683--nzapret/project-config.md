---
trigger: always_on
description: This repository contains the source tree for the Android `nzapret` Magisk/KernelSU module. The module bypasses DPI on Android by:
---

# AGENTS.md

## Project Summary

This repository contains the source tree for the Android `nzapret` Magisk/KernelSU module. The module bypasses DPI on Android by:

- installing an architecture-specific `nfqws2` binary,
- creating IPv4/IPv6 `iptables` and `ip6tables` NFQUEUE rules,
- launching `nfqws2` with arguments from the active profile,
- exposing a shell CLI and a KernelSU WebUI for control and diagnostics.

This is not a conventional app repository. Most behavior lives in shell scripts plus static assets and packaged data files.

## Repository Map

- `module.prop`
  - Module metadata and version source for releases.
- `customize.sh`
  - Install-time extraction, architecture selection, binary rename, and permission setup.
- `common.sh`
  - Shared POSIX `sh` helpers sourced by `service.sh` and `system/bin/nzapret` (logging, network-mode, IPv6 detection, Private DNS, Android settings I/O). Sourced, not executed.
- `service.sh`
  - Main runtime entrypoint for boot/manual start. Rebuilds firewall state and launches `nfqws2`.
- `uninstall.sh`
  - Stop/cleanup logic. Used for uninstall and CLI `stop`.
- `action.sh`
  - Quick toggle action for Magisk/KernelSU. Delegates to the CLI.
- `system/bin/nzapret`
  - Main CLI for lifecycle control, diagnostics, list refresh, profile switching, and JSON output for the WebUI.
- `profiles/*.conf`
  - `nfqws2` argument profiles. The current tree ships `profiles/default.conf`.
- `lists/`
  - Hostlists used by the active profile. `list-user.txt` is shipped as an empty file in the module ZIP.
- `payloads/*.bin`
  - Fake TLS/QUIC payloads referenced by profiles.
- `lua/*.lua`
  - Upstream `nfqws2` helper libraries loaded by profiles via `--lua-init`.
- `bin/nfqws2-*`
  - Architecture-specific binaries. `customize.sh` renames the selected one to `bin/nfqws2` during install.
- `bin/nztg-*`
  - Architecture-specific Telegram MTProto proxy binaries (a static, CGO-free Go port of tg-ws-proxy; sources live on the `nztg` branch). `customize.sh` renames the selected one to `bin/nztg` during install and keeps it (not deleted alongside the other arch binaries).
- `tgproxy.conf`
  - Mutable Telegram proxy config (`host`, `port`, `cf_enabled`, `cf_domain`, repeated `dc=` rules). Created with original defaults on first run; not shipped; preserved across updates.
- `.tg_secret`
  - Mutable MTProto secret (32 hex), owned by `bin/nztg --secret-file`. Not shipped; preserved across updates.
- `webroot/`
  - KernelSU WebUI (`index.html`, `style.css`, `kernelsu.js`).
- `META-INF/com/google/android/*`
  - Installer glue for the flashable module ZIP.
- `profiles/profile.current`
  - Mutable active-profile pointer consumed by runtime and CLI.
- `build.sh`
  - Packaging helper: stages the module, normalizes text line endings to LF, removes runtime artifacts, and builds the ZIP.
- `.github/release-notes/*.md`
  - Versioned release bodies. A release for `module.prop` version `vX.Y.Z` requires `.github/release-notes/vX.Y.Z.md`.
- `.github/workflows/release.yml`
  - Manual GitHub Actions workflow that runs `bash build.sh`, requires the matching versioned release notes file, generates `update.json`, and publishes a release from `module.prop` version.

## Runtime Flow

1. The installer runs `customize.sh`, which unpacks the module, selects `bin/nfqws2-$ARCH`, renames it to `bin/nfqws2`, removes the unused binaries, and fixes permissions.
2. At boot, or via a manual CLI start, `service.sh` waits for Android boot completion, ensures mutable runtime files exist, initializes Android Private DNS once, resolves the network stack mode, loads the active profile, recreates the `nzapret_out` chains in IPv4 and optionally IPv6 `mangle`, and launches `nfqws2`.
3. `system/bin/nzapret` is the operator-facing interface. It wraps start/stop/restart, updates hostlists, switches profiles, manages Android Private DNS, exposes diagnostics, and returns JSON consumed by the WebUI.
4. `webroot/index.html` talks to the CLI through `ksu.exec(...)`; it does not mutate module internals directly.

## Critical Invariants

- Keep runtime scripts compatible with Android `sh`.
  - `service.sh`, `uninstall.sh`, `action.sh`, and `system/bin/nzapret` all use `#!/system/bin/sh`.
  - Avoid bash-only syntax in those files.
  - `build.sh` is the only script intentionally written for bash.

- Treat the installed module path as a coordinated constant.
  - `system/bin/nzapret` hardcodes `MODDIR="/data/adb/modules/nzapret"`.
  - `profiles/default.conf` and `profiles/profile.current` live under `/data/adb/modules/nzapret/profiles/...`.
  - `webroot/index.html` also shells out against `/data/adb/modules/nzapret`.
  - Changing module ID or install path requires synchronized updates across multiple files.

- Keep boot-time behavior local-only.
  - `service.sh` should not download lists or depend on the network.
  - List refresh belongs to `system/bin/nzapret update`.
  - The boot service may read/write local Android global settings for Private DNS, but must guard the `settings` command and must not require external connectivity.

- Profiles are both parsed and passed through.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nikitos4683/nzapret](https://github.com/nikitos4683/nzapret) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
