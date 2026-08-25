---
trigger: always_on
description: Instructions for AI coding agents (Claude Code, Cursor, Copilot, etc.) working in this repository.
---

# AGENTS.md — OpenEmu-Silicon

Instructions for AI coding agents (Claude Code, Cursor, Copilot, etc.) working in this repository.

---

## Read First

Before doing any work, read this file fully. It is the authoritative source for how this project is structured and how changes should be made.

---

## About This Project

OpenEmu-Silicon is a community-maintained fork of OpenEmu, rebuilt to run natively on Apple Silicon (arm64) without Rosetta. It descends from:

- [OpenEmu/OpenEmu](https://github.com/OpenEmu/OpenEmu) — the original project
- [bazley82/OpenEmuARM64](https://github.com/bazley82/OpenEmuARM64) — the foundational ARM64 port

The goal is to honor the original OpenEmu spirit — a beautifully designed, first-class native macOS game emulation frontend — while making it work reliably on M-series Macs with modern macOS and Swift.

**The maintainer is not a professional developer.** If you are writing explanations, commit messages, or comments, please use plain language. Avoid jargon where a plain word works just as well.

---

## Ground Rules

1. **Never commit directly to `main`.** All work goes through feature branches → PRs → `main`.
2. **Branch from `main`, open PRs against `main`.** There is no staging branch.
3. **Build before committing.** Run an `xcodebuild` check on any Swift/ObjC changes before staging a commit.
4. **Don't rewrite files wholesale.** This is a large, complex Xcode project. Make surgical changes. Rewriting `.pbxproj` or large ObjC files without understanding them will break the build.
5. **Respect the flattened architecture.** Submodule directories (`Nestopia/`, `BSNES/`, etc.) are regular directories — do not attempt to re-initialize them as git submodules.
6. **Do not commit build artifacts.** No `.o` files, derived data, `.app` bundles, build logs, or compiled executables.

---

## Language and Tooling

- **Swift 6.3.2** — strict concurrency is enforced. Use `@MainActor`, `Sendable`, and structured concurrency correctly.
- **Objective-C** — many core files are ObjC. Bridge headers are in place. Don't break them.
- **Xcode 26.5** — use `xcodebuild` for CLI builds. The primary workspace is `OpenEmu-metal.xcworkspace`.
- **No package manager** — no SPM, no CocoaPods, no Carthage. Dependencies are vendored or flattened submodules.

---

## Build Command

The canonical verification floor is `Scripts/verify.sh`. It builds, runs the static analyzer, validates Info.plist and entitlements, and checks codesign — a stricter floor than a bare `xcodebuild build`.

```bash
./Scripts/verify.sh                 # build + analyze + plist + codesign
./Scripts/verify.sh --launch        # above + 5s smoke launch with crash check
./Scripts/verify.sh --test          # above + run OpenEmuTests unit target
```

A bare build check is acceptable for quick iteration:

```bash
xcodebuild \
  -workspace OpenEmu-metal.xcworkspace \
  -scheme OpenEmu \
  -configuration Debug \
  -destination 'platform=macOS,arch=arm64' \
  build 2>&1 | tail -30
```

**A clean `verify.sh` run is the definition of "passing."** Run it before every push touching source files. The pre-push git hook in `.githooks/pre-push` enforces this mechanically — install it once per clone with `./Scripts/install-hooks.sh`.

### Setup and Credentials Pre-Requisites

Before compiling for the first time, you must generate the local gitignored secrets/credentials files from their templates:
```bash
cp OpenEmu/ScreenScraperDevCredentials.template.swift OpenEmu/ScreenScraperDevCredentials.swift
cp OpenEmu/OEGoogleDriveSecrets.template.swift OpenEmu/OEGoogleDriveSecrets.swift
```

---

## File Organization

| What you're touching | Where it lives |
|----------------------|---------------|
| Main app logic | `OpenEmu/*.swift` and `OpenEmu/*.m` |
| Shared protocols/types | `OpenEmu-SDK/` |
| UI components | `OpenEmuKit/` |
| Metal shaders | `OpenEmu-Shaders/` |
| Emulator cores | `[CoreName]/` (top-level dirs) |
| Build and utility scripts | `Scripts/` |
| Xcode project | `OpenEmu/OpenEmu.xcodeproj/` |

---

## Supported Cores (as of 2026)

| System | Core(s) |
|--------|---------|
| 3DO | 4DO |
| Arcade | MAME |
| Atari 2600 | Stella |
| Atari 5200 | Atari800 |
| Atari 7800 | ProSystem |
| Atari 8-bit | Atari800 |
| Atari Jaguar | VirtualJaguar |
| Atari Lynx | Mednafen |
| ColecoVision | JollyCV (default), CrabEmu, blueMSX |
| Commodore 64 | (RetroArch / VICE only — no native core ships in this fork) |
| Famicom Disk System | Nestopia |
| Game Boy / GBC | Gambatte |
| Game Boy Advance | mGBA |
| Game Gear | Genesis Plus GX (GenesisPlus) |
| GameCube | Dolphin |
| Intellivision | Bliss |
| MSX | blueMSX |
| Neo Geo Pocket | Mednafen |
| Nintendo (NES) | Nestopia (default), FCEU |
| Nintendo 64 | Mupen64Plus |
| Nintendo DS | DeSmuME |
| Odyssey² / Videopac+ | O2EM |
| PC Engine | Mednafen |
| PC Engine CD | Mednafen |
| PC-FX | Mednafen |
| Pokémon Mini | PokeMini |
| Sega 32X | Picodrive |
| Sega CD / Mega CD | Genesis Plus GX (GenesisPlus) |
| Sega Dreamcast | Flycast |
| Sega Genesis / Mega Drive | Genesis Plus GX (GenesisPlus) |
| Sega Master System | Genesis Plus GX (GenesisPlus) |
| Sega Saturn | Mednafen |
| Sony PlayStation | Mednafen |
| Sony PSP | PPSSPP |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenEmu-Silicon/OpenEmu-Silicon](https://github.com/OpenEmu-Silicon/OpenEmu-Silicon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
