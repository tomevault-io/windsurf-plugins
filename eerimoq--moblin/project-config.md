---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Moblin is a free iOS/iPadOS IRL streaming app (Swift/SwiftUI) targeting Twitch, YouTube, Kick, Facebook
and OBS Studio. It streams over RTMP(S), SRT(LA), RIST and WHIP/WebRTC, with SRTLA/RIST bonding over
multiple network interfaces. The repo also contains an Apple Watch companion, a Live Activity extension,
a home screen widget, a screen recording broadcast extension, and a SolidJS web remote control frontend.

## Commands

```sh
just style           # swiftformat + oxfmt + isort + black (auto-fix)
just style-check     # same, lint-only
just lint            # swiftlint --strict + oxlint + pylint + ruff check + mypy + xcstringslint
just lint-fix        # auto-fix Localizable.xcstrings issues
just spell-check     # codespell
just periphery       # dead code detection (needs full index; slow)

just web-remote-control-frontend-prepare   # npm install
just web-remote-control-frontend-build     # tsc --noEmit + vite build → Moblin/RemoteControl/Web/

just machine-translate                     # fill missing translations in Localizable.xcstrings
```

CI (`.github/workflows/all.yml`) runs `style-check`, `lint`, `spell-check`, the web frontend build followed
by `git diff --exit-code`, and an `xcodebuild build` of the `Moblin` scheme. Two consequences: **the built
web assets in `Moblin/RemoteControl/Web/` are committed and must be regenerated whenever
`WebRemoteControlFrontend/` changes**, and **CI never runs the unit tests** — run them yourself.

### Unit tests

Swift Testing (not XCTest): suites are `struct *Suite` in `MoblinTests/` using `@Test` and `#expect`.

**Always run the unit tests for Mac Catalyst, never for a simulator or a device, when a test run is needed:**

```sh
xcodebuild test -scheme Moblin -destination 'platform=macOS,variant=Mac Catalyst'
```

### System tests

`tests/` holds a Python harness that drives a real device against `mediamtx`/`ffmpeg`. Ask user to 
start the app before running the test commands.

```sh
just test
just test --device macpro Talkback
just test-stability                        # long-running soak test, 12 hours by default
```

The harness runs with `tests/` as the working directory, so its imports are `from utils.moblin import
Moblin`, while `just lint` type checks `tests/` and `utils/` from the repo root. That only works while
every module name is unique across both trees — `tests/suites/` and `tests/utils/` are packages
(`__init__.py`) so that `tests/suites/stability.py` does not collide with the `tests/stability.py` entry
point. Adding a `utils/x.py` that shadows a `tests/x.py` (or dropping an `__init__.py`) makes mypy abort
with "Duplicate module named ..." instead of checking anything.

## Architecture

### Settings vs Model — the central split

Two parallel object graphs, and picking the wrong one is the most common mistake:

- **`Moblin/Various/Settings/`** — persisted user configuration. `Settings` owns a `Database` serialized to
  JSON on disk; `Settings.store()` writes it. Everything here is `Codable`.
- **`Moblin/Various/Model/`** — runtime state (is live, current bitrate, connected devices, active effects).
  Not persisted.

Settings classes are `Codable, Identifiable, ObservableObject` with `@Published` properties and
**hand-written `encode(to:)`/`init(from:)`**. Decoding goes through the helper in
`Common/Various/CommonUtils.swift`:

```swift
name = container.decode(.name, String.self, "")   // never throws; falls back to the default
```

This is what makes old settings files forward-compatible, so a new property needs a `CodingKey`, an
`encode` line and a `decode` line with a sensible default — omitting them silently drops the value on
reload. Enum raw values (`case text = "Text"`) are the persisted representation and must not be renamed;
`toString()` supplies the localized display string instead.

Structural changes that defaults cannot express go in `Settings.migrateFromOlderVersions()`
(`Settings.swift`), which uses per-object `migrated` boolean flags and calls `store()` as it goes.

Secrets are kept out of the JSON: `store()` calls `extractSensitiveData` to null out tokens before
writing and `insertSensitiveData` to restore them, with the real values living in the Keychain
(`Keychain.swift`, `addSensitiveData` on load).

### Model — one class, ~100 extensions

`Model` (`Moblin/Various/Model/Model.swift`) is a single `final class Model: NSObject, ObservableObject`
split across ~60 `ModelXxx.swift` files, each an `extension Model` for one feature area (`ModelChat`,
`ModelTwitch`, `ModelRecording`, …). New feature code belongs in its own `extension Model` file, not in
`Model.swift`.

This codebase uses **`ObservableObject`/`@Published`, not the `@Observable` macro** — there are zero uses
of `@Observable`. Because a single `@Published` change on a class this large would invalidate every
observing view, `Model.swift` declares many small `ObservableObject` "provider" classes (`Bitrate`,
`Bonding`, `Battery`, `StatusTopLeft`, `Toast`, `StreamOverlay`, `SceneSelector`, …). Views observe the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eerimoq/moblin](https://github.com/eerimoq/moblin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
