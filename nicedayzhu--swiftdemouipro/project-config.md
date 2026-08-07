---
trigger: always_on
description: This file applies to the entire repository. It is the operational reference for coding agents working on Swift DemoUI Pro. Keep it aligned with the scripts and architecture whenever those change.
---

# Swift DemoUI Pro Agent Guide

This file applies to the entire repository. It is the operational reference for coding agents working on Swift DemoUI Pro. Keep it aligned with the scripts and architecture whenever those change.

## Project Background

Swift DemoUI Pro is an unofficial, client-side enhancement for Counter-Strike 2 Demo and HLTV playback. It has two cooperating parts:

1. A Panorama override that extends Valve's native `huddemocontroller` with recorded-voice controls, player POV switching, and direct round navigation.
2. A native Qt 6 Widgets launcher that detects CS2, accepts `.dem` files or ZIP archives, installs the override for an isolated playback session, launches CS2 with `-insecure`, and removes the temporary changes afterward.

The project does not require SwiftlyS2, a server plugin, a Workshop item, or a running server. It is not affiliated with or endorsed by Valve or FACEIT.

The launcher must never imply that an `-insecure` session is suitable for matchmaking. It does not change permanent Steam launch options.

## Non-Negotiable Safety Invariants

- Never launch the Demo workflow while CS2 is already running. `-insecure` must be applied when the launcher starts a new process.
- Back up `gameinfo.gi` before the first modification and preserve its detected encoding.
- SearchPath modification must remain exact, idempotent, and reversible. Remove only the line owned by this project.
- Use atomic writes/copies for `gameinfo.gi`, the staged Demo, CFG files, session markers, and installed VPK whenever possible.
- Cleanup must refuse to proceed while CS2 is running and must delete only project-owned paths. Keep the explicit staged-directory suffix check before recursive deletion.
- Preserve the user's unrelated CS2 files, SearchPaths, Steam launch options, settings, and installed overrides.
- Do not run `demo-menu.ps1 -Action Install`, `-Action Uninstall`, or `-InstallLocalOverride` during ordinary compilation/testing unless the user explicitly requests a local CS2 installation change.
- Do not run `release.ps1 -Publish`, create/push tags, push commits, or publish a GitHub Release without explicit user authorization.

## Repository Map

| Path | Responsibility |
| --- | --- |
| `addon/panorama/layout/hud/huddemocontroller.xml` | Native-compatible DemoUI layout override. Preserve required native controls and IDs. |
| `addon/panorama/scripts/hud/swift_demo_voice.js` | Player discovery, 64-slot voice masks, POV validation, and round navigation. Runs in Panorama, not Node.js. |
| `addon/panorama/styles/hud/swift_demo_voice.css` | In-game Demo Voice panel styling. |
| `powershell/` | Shared compile, VPK pack, install, and uninstall implementation. |
| `demo-menu.ps1` | Root entry point for Panorama compile/pack/install lifecycle actions. |
| `launcher/src/Cs2Manager.*` | Steam/CS2 discovery, SearchPath lifecycle, ZIP inspection/extraction, session staging, launch, and cleanup. |
| `launcher/src/LauncherWindow.*` | Qt UI construction, state refresh, dialogs, localization, and user actions. |
| `launcher/src/main.cpp` | Qt application entry point and visual-preview command-line options. |
| `launcher/tests/tst_launcher_core.cpp` | Qt tests for Steam parsing, safe SearchPath edits, isolated sessions, ZIP behavior, and launch arguments. |
| `launcher/third_party/miniz/` | Vendored miniz source and MIT license used for in-process ZIP reading. |
| `launcher/translations/` | Qt Linguist `.ts` translation sources. Generated `.qm` files are build outputs. |
| `tests/test_demo_voice_mask.js` | Node-based Panorama logic and native-layout integration tests. |
| `VERSION` | Single semantic-version source for CMake, UI, Windows resources, and package names. |
| `release.ps1` | Full local release-candidate build and optional GitHub publication entry point. |
| `.github/workflows/ci.yml` | Portable Windows CI for JavaScript and Qt tests; it intentionally does not build the VPK. |
| `README.md`, `README_CN.md` | Concise player-facing English and Simplified Chinese documentation; keep them synchronized. |
| `DEVELOPMENT.md`, `DEVELOPMENT_CN.md` | Developer-facing build, test, localization, versioning, release, and contribution documentation. |
| `docs/images/` | Screenshots referenced by public documentation; keep relative links portable. |

## Architecture and Behavior

### Panorama override

- The override augments the native DemoUI instead of replacing its timeline, playback controls, settings, and camera-mode behavior.
- Display slots 1-32 map to `tv_listen_voice_indices`; slots 33-64 map to `tv_listen_voice_indices_h`. JavaScript bitwise results must remain signed 32-bit console values.
- `BuildMasksForSlots` accepts normalized zero-based slots and produces the low/high masks. Do not introduce off-by-one conversions between display slots and internal slots.
- Player rows are derived from `GameStateAPI` with fallbacks for slot, team, name, and status. Dead/disconnected players may retain voice toggles but must not offer misleading POV actions.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nicedayzhu/SwiftDemoUIPro](https://github.com/nicedayzhu/SwiftDemoUIPro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
