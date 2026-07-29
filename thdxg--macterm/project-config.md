---
trigger: always_on
description: A macOS terminal emulator built with SwiftUI and libghostty. Single-window app with project-based workspace management, split panes, and a quick terminal overlay.
---

# Macterm Codebase Guide

A macOS terminal emulator built with SwiftUI and libghostty. Single-window app with project-based workspace management, split panes, and a quick terminal overlay.

## Build & Run

```bash
mise install          # Install tools (gh, swiftformat, swiftlint, xcodegen, xcbeautify)
mise run setup        # Download pre-built GhosttyKit.xcframework
mise run run          # Build and launch (debug)
mise run logs         # Stream live logs from the debug app (--release for release app)
mise run format       # Auto-fix formatting with swiftformat
mise run lint         # swiftlint
mise run test         # Run the test suite
mise run build        # Release build + DMG
mise run bench        # Release build + window-state resource benchmark
```

`format`, `lint`, and `test` show a spinner and print output only on failure. **Always pass `--verbose`** (e.g. `mise run test --verbose`) to stream the raw output.

Requires macOS 14+, Swift 6.0+. Liquid glass and a few chrome refinements are macOS 26 (Tahoe) features that degrade gracefully on older systems (gated behind `WindowAppearance.glassSupported` / `#available`). GhosttyKit is a pre-built xcframework from `thdxg/ghostty` (a fork that adds CI builds); no zig toolchain needed.

`GhosttyKit.xcframework` and the `Macterm/Resources/{ghostty,terminfo,…}` contents are gitignored artifacts downloaded by `mise run setup` — **every fresh checkout, including a git worktree, must run `mise run setup`** before it can build. Don't symlink them from another checkout: `setup.sh` only re-downloads when the artifact is _absent_ (presence check, not version check), so a symlinked copy silently goes stale. To refresh a stale artifact, delete it and re-run setup.

## Releasing & Updates

Auto-updates ship via [Sparkle](https://sparkle-project.org/) — daily background check, manual via **Macterm → Check for Updates…**. Updates verify an EdDSA signature, so no `xattr` workaround after first install. No telemetry.

Tag-pushed builds release via `.github/workflows/release.yml`, which needs repo secrets: `GH_PAT` (contents:read on `thdxg/ghostty`, downloads GhosttyKit), `SPARKLE_ED_PUBLIC_KEY` (baked into `Info.plist`), and `SPARKLE_ED_PRIVATE_KEY` (signs each DMG — **back it up; losing it means users can't auto-update to any further release**). The workflow appends an `<item>` to `appcast.xml` on `gh-pages` (served at `https://thdxg.github.io/macterm/appcast.xml`, the feed URL in `Info.plist`) along with a per-version notes page rendered from the GitHub Release body (`publish-appcast.sh`).

## Benchmarks


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thdxg/macterm](https://github.com/thdxg/macterm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
