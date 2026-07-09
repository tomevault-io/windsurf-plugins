---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

HapticBreak is a macOS (13+) menu-bar break reminder that notifies via **trackpad haptics** (Taptic Engine) instead of pop-ups or sounds. Pure Swift (SPM executable, AppKit + SwiftUI), no Dock icon (`LSUIElement`). Only dependency: Sparkle (in-app updates).

## Commands

```bash
swift build                        # debug build
swift test                         # all unit tests
swift test --filter BreakTimerTests            # one test class
swift test --filter BreakTimerTests/testInitialCountdown   # one test method

./build.sh release                 # assemble build/HapticBreak.app (ad-hoc signed)
./build.sh release dmg             # also produce a distributable .dmg
scripts/release-check.sh           # full pre-release gate: build → tests → CLI smoke → UI snapshots → package
scripts/release-check.sh --no-package   # validation only, faster
```

The binary itself has headless/diagnostic entry modes (see `main.swift`), useful for verification without launching the full app:

```bash
swift run HapticBreak --logictest        # headless BreakTimer state-machine self-test (CI-safe)
swift run HapticBreak --checkenv         # read-only probe of idle/fullscreen/Focus/mic/stats (no writes)
swift run HapticBreak --rendershots DIR [zhHans|en|ja]  # offscreen-render UI snapshots for visual review
swift run HapticBreak --selftest         # verify haptics actually fire (needs real trackpad)
swift run HapticBreak --presets [1-10]   # feel every built-in pattern back-to-back
swift run HapticBreak --hapticlab        # calibration bench window
swift run HapticBreak --demo             # full app on ephemeral storage
```

Haptics only exist on real hardware (Force Touch trackpad / Magic Trackpad 2+); anything "does it feel right" must be auditioned on a real Mac — tests and `--rendershots` cover logic and UI only.

## Release process

### Pre-release checklist

1. **Write `CHANGELOG.md`** — add a `## [X.Y.Z] - YYYY-MM-DD` section under `[Unreleased]`; update the comparison links at the bottom.
2. **Bump version** — change the `VERSION` default in `build.sh` (line ~13) from the old version to the new one.
3. **Commit** — `git add -A && git commit -m "release: X.Y.Z — <one-line summary>"`.
4. **Local gate** — run the full pre-release check (builds Universal binary, runs all tests, CLI smoke, UI snapshot walk, and packages .app + .dmg):
   ```bash
   scripts/release-check.sh --universal --dmg
   ```
5. **Tag & push** — triggers the CI pipeline:
   ```bash
   git tag vX.Y.Z && git push origin main --tags
   ```
6. **(Post-release) Homebrew Cask** — update `packaging/homebrew/Casks/hapticbreak.rb`: bump `version`, update `sha256` from the released `.dmg`.

### What CI does automatically (`release.yml`)

Push tag `vX.Y.Z` → `swift test` → `build.sh release dmg universal` (Universal arm64+x86_64) → `--logictest` smoke → (if signing secrets configured) Developer ID codesign + notarization + staple → collect `.dmg` + `.zip` → extract matching CHANGELOG section → create GitHub Release → (if `UPDATE_PRIVATE_KEY` configured) `generate_appcast` → push `appcast.xml` + site to `gh-pages`.

### Signing & auto-update secrets (optional, in repo Settings → Secrets)

| Secret | Purpose |
|--------|---------|
| `MACOS_CERTIFICATE_BASE64` | Developer ID Application `.p12` (base64) |
| `MACOS_CERTIFICATE_PASSWORD` | Password for the `.p12` |
| `MACOS_SIGN_IDENTITY` | e.g. `"Developer ID Application: Name (TEAMID)"` |
| `APPLE_ID` / `APPLE_TEAM_ID` / `APPLE_APP_PASSWORD` | Notarization credentials |
| `UPDATE_PRIVATE_KEY` | EdDSA private key for Sparkle appcast signing |

Without signing secrets the pipeline still produces a working (ad-hoc signed) Release; without `UPDATE_PRIVATE_KEY` it skips appcast generation. See `packaging/autoupdate/README.md` for one-time setup.

### Version numbering

Follows [Semantic Versioning](https://semver.org/): `MAJOR.MINOR.PATCH`. The build number (`CFBundleVersion`) is auto-derived (`1.1.0` → `10100`) and must be monotonically increasing for in-app updates to work.

Homebrew cask lives in `packaging/homebrew/Casks`; `site/` is the GitHub Pages promo site.

## Architecture

**Entry & flow control.** `main.swift` is a dispatcher: each `--flag` above is a standalone headless entry; with no flags it launches `AppDelegate` → `AppController`. `AppController` (Core/) is the orchestration hub — it owns Settings, the timer, haptics, statistics, menu bar and windows, and is the only place they're wired together.

**Timer state machine.** `BreakTimer` (Core/BreakTimer.swift) is a pure, dependency-free state machine (`working → reminding → resting`, pause reasons, typing-aware defer, heads-up, nudge cap → auto-postpone). It owns no timers: `AppController` ticks it once per second and reacts through `BreakTimerDelegate`. This is what makes it testable headlessly — both in `Tests/` and via `--logictest` (`Sources/HapticBreak/LogicTests.swift`, compiled into the app on purpose).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tmpbin/HapticBreak](https://github.com/tmpbin/HapticBreak) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-09 -->
