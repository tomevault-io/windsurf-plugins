---
trigger: always_on
description: This repository adapts the official macOS Codex Desktop DMG to a runnable Linux build, packages that build as native `.deb`, `.rpm`, and pacman artifacts, and ships a local Rust update manager that rebuilds future Linux packages from newer upstream DMGs.
---

# AGENTS.md

## Purpose

This repository adapts the official macOS Codex Desktop DMG to a runnable Linux build, packages that build as native `.deb`, `.rpm`, and pacman artifacts, and ships a local Rust update manager that rebuilds future Linux packages from newer upstream DMGs.

The current working flow is:

1. `install.sh` extracts `Codex.dmg`
2. extracts and patches `app.asar`
3. rebuilds native Node modules for Linux
4. downloads a Linux Electron runtime
5. writes a Linux launcher into `codex-app/start.sh`
6. `scripts/build-deb.sh`, `scripts/build-rpm.sh`, or `scripts/build-pacman.sh` packages `codex-app/`
7. `codex-update-manager` runs as a `systemd --user` service and manages local auto-updates

## Source Of Truth

- `install.sh`
  Main installer and launcher generator.
- `scripts/build-deb.sh`
  Builds the `.deb` from the already-generated `codex-app/`.
- `scripts/build-rpm.sh`
  Builds the `.rpm` from the already-generated `codex-app/`.
- `scripts/build-pacman.sh`
  Builds the `.pkg.tar.zst` from the already-generated `codex-app/`.
- `scripts/install-deps.sh`
  Installs host dependencies and bootstraps Rust.
- `scripts/lib/package-common.sh`
  Shared shell helpers used by the native package builders.
- `Makefile`
  Convenience targets for build, package, install, and cleanup workflows.
- `packaging/linux/control`
  Debian control template.
- `packaging/linux/codex-desktop.desktop`
  Desktop entry template.
- `packaging/linux/codex-packaged-runtime.sh`
  Packaged-launcher helper for native-package-only runtime behavior.
- `packaging/linux/codex-desktop.spec`
  RPM spec template.
- `packaging/linux/codex-update-manager.service`
  User-level `systemd` unit for the local update manager.
- `packaging/linux/codex-update-manager.prerm`
  Debian maintainer script that stops or disables the user service during removal.
- `packaging/linux/codex-update-manager.postrm`
  Debian maintainer script that reloads affected user managers after removal.
- `assets/codex.png`
  App icon used in native packages.
- `updater/`
  Rust crate that checks for new upstream DMGs, rebuilds local native-package artifacts, tracks update state, and installs prepared packages after the app exits.
- `updater/Cargo.toml`
  Source of truth for the updater crate version and dependency policy.
- `docs/webview-server-evaluation.md`
  Decision record for the future Python-to-Rust webview server discussion.

## Generated Artifacts

- `codex-app/`
  Generated Linux app directory. Treat this as build output unless you are intentionally patching the launcher or testing package contents.
- `dist/`
  Generated packaging output, including `dist/codex-desktop_*.deb`, `dist/codex-desktop-*.rpm`, and `dist/codex-desktop-*.pkg.tar.zst`.
- `Codex.dmg`
  Cached upstream DMG. Useful for repeat installs.
- `~/.config/codex-update-manager/config.toml`
  Runtime config written or read by the updater service.
- `~/.local/state/codex-update-manager/state.json`
  Updater state machine persistence.
- `~/.local/state/codex-update-manager/service.log`
  Updater service log.
- `~/.cache/codex-update-manager/`
  Downloaded DMGs, rebuild workspaces, staged package artifacts, and build logs.

Do not assume `codex-app/` is pristine. If behavior differs from `install.sh`, prefer updating `install.sh` and then regenerating the app.

## Important Behavior And Known Fixes

- DMG extraction:
  `7z` can return a non-zero status for the `/Applications` symlink inside the DMG. This is currently treated as a warning if a `.app` bundle was still extracted successfully.
- Launcher and `nvm`:
  GUI launchers often do not inherit the user's shell `PATH`. The generated `start.sh` explicitly searches for `codex`, including common `nvm` locations.
- CLI preflight:
  Before Electron launches, the generated launcher asks `codex-update-manager` to verify the installed Codex CLI and update it if the npm package is newer. The check is best-effort: it uses a 1-hour cooldown for npm registry lookups, falls back to `npm install -g --prefix ~/.local` if a global install fails, and warns instead of blocking app launch when the refresh attempt does not succeed.
- Linux file manager integration:
  During ASAR patching, `scripts/patch-linux-window-ui.js` also tries to inject a Linux implementation for `Open in File Manager`. The patch is intentionally fail-soft: if the upstream minified bundle no longer matches, the install continues and emits exactly `Failed to apply Linux File Manager Patch`.
- Linux translucent sidebar default:
  During the same ASAR patch step, Linux defaults `Translucent sidebar` to `false` by applying `opaqueWindows: true` only when the app has no saved explicit value yet. This keeps existing user preferences intact while avoiding the sidebar disappearing bug on first run.
- Launcher logging:
  The generated launcher logs to:
  `~/.cache/codex-desktop/launcher.log`
- App liveness:
  The launcher writes a PID file to `~/.local/state/codex-desktop/app.pid`. The updater uses that plus `/proc` fallback to know whether Electron is still running.
- Desktop icon association:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ilysenko/codex-desktop-linux](https://github.com/ilysenko/codex-desktop-linux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
