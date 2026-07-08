---
trigger: always_on
description: > Read this file first, every session. Then read `STATUS.md` to find the current task.
---

# CLAUDE.md — Silo operating manual

> Read this file first, every session. Then read `STATUS.md` to find the current task.
> This file is the contract; `STATUS.md` is the live state.

## Mission
**Silo** is a native macOS (SwiftUI) launcher overlay for Windows Steam games run via Wine + Apple's
Game Porting Toolkit (GPTK / D3DMetal). Topology = **Single Downloader, Multi-Runtime**:
- Steam is installed **once** into a single *simple* Master Wine bottle, used only to download games.
- Each game is **launched in its own isolated Wine prefix** with its own graphics backend + env.

Pipeline: **Discovery** (parse `appmanifest_*.acf`) → **Provision** (seed per-game prefix) →
**Graphics Linker** (inject GPTK/D3DMetal, CrossOver fallback) → **Launch Orchestrator** (detached
process with `WINEPREFIX` overridden to the isolated prefix).

## Hard constraints (non-negotiable)
1. **SwiftPM only — never call `xcodebuild`.** This machine has Command Line Tools only (no Xcode).
   Build with `swift build`; the `.app` is assembled by `Scripts/build-app.sh`.
2. **Swift 6 strict concurrency** (`swiftLanguageMode(.v6)`). No `@unchecked Sendable` to silence
   errors — derive correct isolation from the concurrency model below. (`@unchecked` is allowed only
   in test doubles where it is genuinely safe and commented.)
3. **NOT App-Sandboxed.** `Resources/silo.entitlements` must never contain
   `com.apple.security.app-sandbox`. The app executes `wine` *outside* its bundle and reads/writes
   `~/Library/Application Support` + the Steam bottle — impossible under the sandbox.
4. **Builds, tests, launches, and parses a library with ZERO runtimes installed.** Wine/GPTK/Steam
   are absent on this machine. Everything runtime-dependent sits behind a resolver returning
   `.notConfigured` → a UI state, never a crash. Tests must pass on a clean machine.
5. **No Homebrew / system package-manager dependency.** Fully self-sustained. The app downloads its
   own Wine/GPTK runtime from a *configurable third-party GitHub release* (Heroic-style) and
   self-updates from GitHub Releases.
6. **No external SPM dependencies.** Updater + runtime downloads use `URLSession` + the GitHub API
   directly. Keep `Package.swift` dependency-free.
7. **Never bundle or auto-download Wine, GPTK, or any Steam-API emulator (Goldberg).** The runtime is
   fetched only from a URL the user can see/override; the emulator stub is **user-provided only**,
   with a prominent legal/ToS caveat, original DLL backed up.
8. **The Wine runtime is built ONLY from CrossOver's FOSS source** (`crossover-sources-<ver>.tar.gz` via
   `Scripts/build-wine.sh` / `build-wine.yml`). This is the ONE accepted base. **Do NOT propose, switch to,
   or suggest** Gcenx/`macOS_Wine_builds` (stale, unverifiable source provenance), Whisky, mainline/staging
   prebuilts, or using an installed CrossOver/CodeWeavers product. Every black-window / login / graphics
   problem is to be **fixed on this from-source CrossOver-FOSS Wine** — debug the build flags, Wine
   registry, env, and Silo's launch code; never answer "use a different runtime." Decided 2026-06-28.

## Graphics backends (GPTK + DXMT — decided 2026-06-30, reverses the GPTK-only stance)
Two Metal translation layers, selectable **per game**: **GPTK / D3DMetal** (Apple's, D3D10/11/12 → Metal,
the default) and **DXMT** (3Shain's, D3D10/11 → Metal directly, the fallback for titles GPTK's
device-creation can't run, e.g. Overcooked 2). DXMT `v0.72` — the **exact version CrossOver 26 bundles** —
is built from its upstream (`3Shain/dxmt`, pinned in `versions.env`) **against the CrossOver Wine**, the
DXMT↔Wine pairing CrossOver itself ships — via `Scripts/build-dxmt.sh` / `.github/workflows/build-dxmt.yml`
(needs full Xcode's Metal toolchain + the wine install for `winemetal.so`). Constraint #8 binds **Wine**
only (DXMT isn't Wine); we build from upstream for the canonical, reproducible build incl. its git
submodules. Never a third-party prebuilt. DXVK was evaluated and rejected (Vulkan/MoltenVK stack; DXMT is
Metal-direct).

**The deterministic rule — backend ⇔ runtime ⇔ bottle** (`GraphicsBackend` is the single source of truth):
- Both backends overlay a **builtin** `d3d11`/`dxgi` into a runtime's `lib/wine` tree, so they can't share
  one runtime. `RuntimeVariants` prepares each: GPTK overlays the base runtime in place (the proven path,
  unchanged); DXMT gets an **APFS clone** of the base + `GraphicsLinker.overlayDXMT`.
- `BottleResolver` is the ONE place that maps `(game, backend) → {prefix, wineBinary, graphics}`. Every
  launch/provision/tool path routes through it — never hard-code `paths.steamBottle` or
  `backend.wineBinaryPath`. A launch emits exactly that backend's `WINEDLLOVERRIDES` builtin set, so it can
  never cross GPTK↔DXMT or silently land on wined3d (it refuses an unconfigured secondary backend).
- **Steam game backend = its bottle.** Each backend gets its own shared Steam bottle (`SteamBottle` for
  GPTK, `SteamBottle-DXMT` for DXMT) — a separate Steam install/login, since one Steam client per prefix.
  A Steam game's backend is *discovered* from which bottle it's installed in (modern → GPTK, older → DXMT).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mikaelhug/Silo](https://github.com/mikaelhug/Silo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
