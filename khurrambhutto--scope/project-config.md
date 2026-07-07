---
trigger: always_on
description: Scope is a desktop app for Linux — inspired by [Mole](https://github.com/tw93/Mole) for macOS — that gives users a single place to see, update, and uninstall every app on their system regardless of how it was installed.
---

# Scope Agent Notes

## What Is Scope

Scope is a desktop app for Linux — inspired by [Mole](https://github.com/tw93/Mole) for macOS — that gives users a single place to see, update, and uninstall every app on their system regardless of how it was installed.

Linux distributions spread packages across APT, Snap, Flatpak, AppImage, and manual installs. Users have no unified view and must remember which tool installed what. Scope solves that.

**Target platform:** Ubuntu (`.deb` first), with plans to support other Debian-based distros and eventually Fedora/Arch families.

## Architecture

- **Frontend:** React + TypeScript in `src/`.
- **Desktop shell:** Tauri v2 + Rust in `src-tauri/`.
- **Package scanning & operations:** Rust backend in `src-tauri/src/`.
- **Package scanning logic:** source-specific modules under `src-tauri/src/scanner/`.
- **Package/app models:** shared typed models in `src-tauri/src/package.rs`.
- **Website / docs:** static files in `docs/`.

Do not restore the old Rust TUI architecture. Scope is now a root Tauri v2 desktop app.

## Modular Codebase Rules

Scope must be developed as a modular app. Do not put feature logic directly in `App.tsx`, `lib.rs`, `main.rs`, or one large catch-all file.

Backend code should be split by responsibility:

- `src-tauri/src/commands/` — small typed Tauri command handlers only.
- `src-tauri/src/package.rs` — shared package/app models and source enums.
- `src-tauri/src/scanner/` — package-source scanners such as APT/dpkg, Snap, Flatpak, AppImage, and future package managers.
- `src-tauri/src/desktop_entries/` — Linux `.desktop` entry discovery and parsing.
- `src-tauri/src/icons/` — Linux icon theme lookup, icon resolution, icon protocol support, fallbacks, and caching.
- `src-tauri/src/operations/` — update/uninstall preview and apply flows.
- `src-tauri/src/safety/` — protected packages, protected paths, stale-plan checks, and backend validation.
- `src-tauri/src/system/` — shared command execution, timeouts, environment detection, and process helpers.

`lib.rs` should wire modules, manage Tauri state, register commands, and start Tauri. It must not contain scanner, icon, update, uninstall, or UI-specific logic.

Frontend code should be split by feature:

- `src/app/` — app shell, layout, providers, and top-level composition.
- `src/features/packages/` — unified installed package list, package rows/cards, filters, search, and package details.
- `src/features/apps/` — GUI app metadata and icon presentation when needed.
- `src/features/updates/` — update preview and update flow.
- `src/features/uninstall/` — uninstall preview and confirmation flow.
- `src/shared/api/` — typed Tauri `invoke` wrappers.
- `src/shared/components/` — reusable UI components.
- `src/shared/types/` — TypeScript models matching backend DTOs.

`App.tsx` should only compose the main shell. It must not contain scanner calls, command strings scattered inline, large UI sections, or business logic.

Every new feature must enter through an appropriate module boundary. If a feature does not clearly belong in one of the existing modules, add a small focused module rather than expanding a large file.

## Current Phase — Unified Package View & Uninstall

Phase one (unified package view with icon resolution) and the uninstall capability are complete. The current focus is phase two remainder: make package **updates** work correctly (preview + apply) before building whole-system cleanup.

The active product surface covers three capabilities:

### 1. See Installed Packages And Apps In One Place

Scope is not only a launcher. It should show packages and apps from supported package sources, including GUI apps and CLI tools, while keeping the main uninstall surface focused on user-relevant packages. Do not surface low-level OS dependencies or package-manager runtimes as normal uninstall targets.

Scan every package source and display a unified package list:

| Source   | How to list installed packages                  |
| -------- | ----------------------------------------------- |
| APT/dpkg | `apt-mark showmanual`, then `dpkg-query` for those package names |
| Snap     | `snap list`, excluding runtime/base snaps       |
| Flatpak  | `flatpak list --user --app --columns=application,name,version,origin,size,description` and `flatpak list --system --app --columns=application,name,version,origin,size,description` |
| AppImage | Scan `~/Applications/` and `~/.local/bin/` for `.AppImage` files |

Each entry should show the best available metadata: name, package id, version, source, installed size, description, update status, protection status, and uninstall capability.

Current scanner product choices:

- APT intentionally reports manually installed packages only. This avoids filling the app with auto-installed dependencies and OS internals that users should not treat as normal apps.
- Snap intentionally hides runtime/base snaps such as `core*`, `snapd`, `bare`, `gtk-*`, and `gnome-*`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [khurrambhutto/scope](https://github.com/khurrambhutto/scope) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
