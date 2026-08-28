---
trigger: always_on
description: macOS menu bar app (SwiftUI + AppKit hybrid). Shows meetings from macOS Calendar / Exchange and fires a full-screen overlay alert at meeting start. Distributed via App Store.
---

# Slapss — Claude Project Context

macOS menu bar app (SwiftUI + AppKit hybrid). Shows meetings from macOS Calendar / Exchange and fires a full-screen overlay alert at meeting start. Distributed via App Store.

Two repos:
- `slapss-app` — macOS app (this repo). **Public**, Apache-2.0. See `CONTRIBUTING.md`.
- `slapss-web` — marketing site on Cloudflare. **Private**, not published. The user-facing changelog is served from it at <https://slapss-app.com/changelog.html>.

`CHANGELOG.md` in this repo is the source of truth for user-facing release notes; `changelog.html` and GitHub Releases are copied from it. The "Changelog log" at the bottom of this file is the separate engineering record — different audience, keep both.

---

## Rules

- **Never bump version numbers without asking Can first.** He decides the version.
- **Release process is in `RELEASING.md`.** Follow it in order; the ordering is what keeps the App Store, the tag, the GitHub Release, and the marketing site in step.
- **After every user-visible change, update `CHANGELOG.md` first**, then mirror it into `slapss-web/changelog.html` (separate private repo). Record the engineering detail in the Changelog log at the bottom of this file. No exceptions.
- Never touch `slapss-web` unless the change requires updating privacy/terms or the changelog.

---

## Architecture

### Entry point
`slapssApp.swift` — `@main`. Creates all `@StateObject`s and injects them as `environmentObject` into every scene.

### Core objects (all `ObservableObject`, injected via environment)
| Object | Responsibility |
|---|---|
| `CalendarAggregator` | Merges EventKit + Microsoft Graph sources, publishes `upcomingMeetings` |
| `AppSettings` | All user preferences, persisted to UserDefaults |
| `AlertScheduler` | Timers, watchdog, App Nap prevention, fires/queues overlays |
| `LocalizationManager` | Runtime language switching without restart |
| `PopoverVisibilityMonitor` | Tracks real popover open/close via NSWindow notifications |

### Key files
- `ContentView.swift` — popover UI (hero card, agenda, `BlobsBackground`, `FloatingDotsBackground`)
- `AlertView.swift` — full-screen overlay card
- `OverlayWindowController.swift` — manages the screen-saver-level `NSWindow`(s) hosting `AlertView`
- `AlertScheduler.swift` — all scheduling logic, App Nap, watchdog, missed-fire recovery
- `CalendarAggregator.swift` — merge + poll loop
- `EventKitSource.swift` — EventKit fetch (Calendar + Reminders)
- `GraphSource.swift` — Microsoft Graph fetch (Exchange)
- `AppSettings.swift` — all `@Published` preferences + UserDefaults persistence
- `PopoverVisibilityMonitor.swift` — NSWindow notification observer
- `Theme.swift` — `AppTheme` (sunset/ocean/forest) + `AppTheme.Accents` color sets + `ThemeSwatchPicker` (shared by Settings and Onboarding)

---

## Non-obvious patterns and gotchas

### The project requires Xcode 26+ — older Xcode fails with actor-isolation errors

`project.pbxproj` sets `SWIFT_APPROACHABLE_CONCURRENCY = YES` and
`SWIFT_DEFAULT_ACTOR_ISOLATION = MainActor` (Swift 6.2 / Xcode 26 settings), with
`SWIFT_VERSION = 5.0`. Every declaration is therefore implicitly `@MainActor`,
which is why so little of this codebase carries explicit `@MainActor` annotations
despite being almost entirely UI code.

Xcode versions older than 26 do not recognise those build settings. They don't
error on the unknown setting — they **silently ignore** it, compile everything as
nonisolated, and then emit a wall of *"call to main actor-isolated instance
method ... in a synchronous nonisolated context"* errors in `MeetingEvent.swift`,
`StatusMenuController.swift`, and `AlertView.swift`. The code is not broken; the
toolchain is too old. This is what broke the first public CI run on a `macos-15`
runner (Xcode 16).

Consequences:
- CI must run on `macos-26` or newer. See `.github/workflows/build.yml`.
- Don't "fix" those errors by sprinkling `@MainActor` or `nonisolated` around.
  Check the Xcode version first.
- If the implicit-MainActor default is ever turned off, the annotations it was
  standing in for have to be added back by hand across the whole codebase.

### MenuBarExtra(.window) keeps the view graph alive permanently
`onAppear` fires once on first popover open. `onDisappear` **never fires** on popover close — the window hides, it is not destroyed. Consequences:
- Never use `onAppear/onDisappear` to gate `.repeatForever()` animations. They will run at 60 fps indefinitely with nothing on screen.
- Use `PopoverVisibilityMonitor.isVisible` instead, which listens to real `NSWindow` key/close notifications.

### Color.clear flips NSView coordinate system on macOS
In a `ZStack` on macOS, `Color.clear`'s NSView backing can leak a flipped coordinate context into sibling views, mirroring glyphs. Always use `.frame(maxWidth: .infinity, maxHeight: .infinity)` to expand a transparent area. Never use `Color.clear` as a spacer in a ZStack.

### Timer.scheduledTimer is App Nap-vulnerable

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theshiver/slapss-app](https://github.com/theshiver/slapss-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
