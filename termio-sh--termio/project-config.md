---
trigger: always_on
description: This file provides guidance to AI coding agents working in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents working in this repository.

## Core rules

- termio is a deliberately small, focused tool. Keep the surface area minimal;
  prefer clarity over cleverness. Do not add features that were not requested.
- Keep changes scoped to the request. Do not refactor unrelated code.
- Respect existing worktree changes. Do not revert the user's edits unless asked.
- Prefer editing existing files over creating new ones. Do not add new
  documentation files unless requested.
- Commit or push only when the user asks.
- Never add an SPM dependency that ships resources — see "Dependencies" below.
  This has shipped as a release-only crash twice.
- Read `CONTRIBUTING.md` for the long-form version of the build, branching, and
  release flow. This file is the short, agent-facing subset.

## Repo overview

termio is a native macOS terminal app for AI coding agents: Swift +
AppKit/SwiftUI on top of **libghostty** (Ghostty's terminal core). It is free
software with no backend, no account, and no license server.

- `Sources/termio` — the macOS app (SwiftPM executable target). Organized by
  feature: `Terminal`, `Sidebar`, `TermioStore`, `Agents`, `Companion`, `Git`,
  `FileBrowser`, `Editor`, `Issues`, `Settings`, `Welcome`, `Info`,
  `CommandPalette`, `Keybindings`, `Browser`, `Theme`, `App`.
- `Shared/` — SwiftPM package (`TermioShared`) shared by the Mac app and the iOS
  companion. The companion wire protocol lives here so both ends stay in sync.
- `ios/` — the iOS companion app (`TermioMobile.xcodeproj`, scheme
  `TermioMobile`).
- `web/landing` — the marketing site (Next.js + Tailwind + shadcn/ui), deployed
  to Vercel.
- `scripts/` — `build-app.sh` (packages the `.app`), `termio` (the CLI that
  drives a running app over its control socket), plus icon and stats helpers.
- `packaging/` — `Info.plist`, entitlements, icon assets for the bundle.
- `docs/` — design docs, RFCs, runbooks, bug handoffs, essays. See "Docs" below.
- `skills/` — agent skills. See "Skills" below.

## Setup

macOS 14+ and Swift 6 (Xcode 26). No `zig` toolchain is needed: libghostty
ships as a prebuilt `GhosttyKit.xcframework` through the
[jiweiyuan/libghostty-swift](https://github.com/jiweiyuan/libghostty-swift)
package. Do not try to build Ghostty from source in this repo.

## Common commands

```sh
swift build                                # resolve + compile
swift run                                  # launch the bare binary
swift test                                 # run the unit tests
./scripts/build-app.sh                     # ad-hoc-signed .app → ./termio.app
TERMIO_CHANNEL=dev ./scripts/build-app.sh  # side-by-side dev app → ./termio-dev.app
./ios/dev-run.sh                           # build + install iOS app on the device
```

termio is a real foreground AppKit app bootstrapped by an explicit
`NSApplication` in `Sources/termio/App/App.swift`, not the SwiftUI `App`
lifecycle. Run it from a macOS GUI session.

Use the dev channel whenever a released termio is installed: it gets its own
bundle id (`sh.termio.app.dev`), state dir (`~/.termio-dev`), companion port
(8788), and `termio-dev` CLI, with Sparkle stripped so it can never auto-update
itself onto the release channel.

The `macos-rebuild-dev` and `ios-rebuild-dev` skills wrap the rebuild-and-
relaunch loop; prefer them over hand-rolling the commands.

## Validation workflow

- `swift build` is the baseline check for any Swift change.
- `swift test` for changes to the units that have coverage — split-tree layout,
  OSC parsing, stall probing, Markdown/HTML rendering, git service, editor text.
- Behavior that only shows up on screen (layout, spacing, focus, terminal
  repaint) needs a real run. Rebuild the dev app, then use the
  `app-screenshot-debug` skill to capture and read back the window.
- Do not screenshot the app yourself by other means, and never claim a UI change
  works without having seen it.
- Task notifications never fire from a dev build. Verify those on a release
  build only.

## Architecture notes

Terminal core:

- libghostty exposes two backends: `.exec` runs a PTY inside ghostty;
  `.inMemory` is host-managed. termio uses **`.inMemory`** — the app owns the
  PTY via `Sources/termio/Terminal/Ghostty/PTYProcess.swift` and the surface only
  renders.
- The PTY is spawned with `forkpty` (login_tty shape). Do **not** switch to
  `posix_spawn`: that shape breaks agents' resize repaint. See
  `docs/bug/terminal-resize-no-reflow-HANDOFF.md`.
- PTY writes must stay non-blocking. A blocking write under the surface lock
  beachballs the app.
- One `TerminalViewState` (`Sources/termio/App/Models.swift`) owns one surface.
  `TermioStore`'s SurfaceCache keeps it alive across view rebuilds so shells
  survive session switching.
- Changes to libghostty itself go to the fork as rebased patch files, not here;
  termio then bumps the package version in `Package.swift`.

State and sessions:

- `TermioStore` is the app's store, split across `TermioStore+*.swift` by
  concern. `StateFile` owns the on-disk snapshot — only the session tree,
  selection, and inspector layout persist. Live state is never written; shells
  restart fresh.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [termio-sh/termio](https://github.com/termio-sh/termio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
