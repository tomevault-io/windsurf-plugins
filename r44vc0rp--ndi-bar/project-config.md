---
trigger: always_on
description: - Repository guidance for agentic coding tools working in ndi-bar.
---

# AGENTS.md

## Purpose
- Repository guidance for agentic coding tools working in ndi-bar.
- Project is a macOS menu bar NDI screen sender (SwiftUI + AppKit).
- Primary target: `ndi-bar` (no test target yet).

## Repository Layout
- `project.yml` — xcodegen manifest. Single source of truth for the Xcode project.
- `ndi-bar/` — Swift sources, entitlements, Info.plist, Assets.xcassets.
  - `NDI/` — Runtime loader and C-struct mirrors for libndi.dylib.
  - `Capture/` — ScreenCaptureKit enumeration and per-display streaming.
  - `State/` — `StreamingController` (the @MainActor ObservableObject).
  - `Menu/` — NSStatusItem/NSMenu controller.
  - `UI/` — SwiftUI Settings view.
- `Makefile` — `make gen | open | build | release | run | clean`.

## Tooling / Environment
- Xcode 16+ required (Swift 5.10, macOS 14+ deployment target).
- `xcodegen` required to materialize `ndi-bar.xcodeproj`. Install with
  `brew install xcodegen`; then `make gen`.
- `libndi.dylib` is loaded at runtime via `dlopen`. The build does NOT link
  against the NDI SDK, so the project compiles even without the SDK
  installed, but will refuse to stream until `/Library/NDI SDK for Apple/`
  exists. Do not add the NDI SDK to the Xcode project file.

## Build Commands
- Generate project:      `make gen`
- Open in Xcode:         `make open`
- Debug build (CLI):     `make build`
- Release build (CLI):   `make release`
- Launch .app:           `make run`
- Clean:                 `make clean`

Raw xcodebuild equivalents:
```sh
xcodebuild -project ndi-bar.xcodeproj -scheme ndi-bar \
  -configuration Debug -destination 'platform=macOS' build
```

## Release / Distribution
- Default local builds are ad-hoc signed (`CODE_SIGN_IDENTITY: "-"` in
  `project.yml`). `make install` and `make run` use this. Good for dev,
  not good for handing to anyone else.
- For a signed+notarized release go through `make dist`. Required env:
  ```sh
  TEAM_ID=ABCDE12345 make dist
  ```
  Signing identity defaults to `Developer ID Application`. Notary
  credentials are loaded from the keychain profile in `NOTARY_PROFILE`
  (default: `NDIBAR_NOTARY`). Store them once via `make notary-login`.
- `make dist` does: sign Release build → verify codesign → zip → notarize
  via `xcrun notarytool submit --wait` → staple ticket → re-zip → sha256.
- Do NOT commit a Team ID to `project.yml`. Keep it in env.
- Do NOT bundle `libndi.dylib` inside the .app. Users install the NDI SDK
  themselves — required by the NDI license and keeps the app small.

## Test Commands
- No test target yet. When added, follow the Dictate pattern:
  ```sh
  xcodebuild -project ndi-bar.xcodeproj -scheme ndi-bar \
    -destination 'platform=macOS' test
  ```

## Lint / Static Checks
- No dedicated linter configured. Treat `xcodebuild` warnings as the primary
  static-check signal. Do not add linters unless explicitly requested.

## Architecture Notes (Important)
- `NDIBarApp.swift` is the `@main` entry point; it uses
  `@NSApplicationDelegateAdaptor` because menubar apps still need
  `NSStatusItem` wiring via AppKit.
- `AppDelegate` is `@MainActor` and owns the `StreamingController` plus the
  `StatusMenuController`.
- `StreamingController` (`@MainActor`, `ObservableObject`) coordinates:
  - NDI runtime loading
  - display enumeration (`SCShareableContent` + `NSScreen.localizedName`)
  - start/stop lifecycle per-display
  - persisted preferences (`UserDefaults`: sourcePrefix, fps, limitTo1080p,
    showsCursor, captureAudio)
  - screen-recording permission gate
- `DisplayStreamer` is NOT @MainActor — SCStream delivery callbacks must
  never touch the main actor on the hot path. It uses two dedicated serial
  queues (`captureQueue`, `audioQueue`).
- `NDILibrary` is a singleton that dlopen's libndi.dylib. All NDI C entry
  points are resolved via `dlsym` and stored as `@convention(c)` function
  pointers in `NDITypes.swift`. Do not introduce a bridging header.
- `NDISender` is RAII — destroying it calls `NDIlib_send_destroy`. One
  sender instance per captured display; sender lifetime is tied to the
  `DisplayStreamer` that owns it.

## Code Style Guidelines

### Imports
- Apple frameworks first, then third-party modules (none right now).
- One import per line.
- Avoid unused imports.

### Formatting
- 4-space indentation, no tabs.
- Match existing brace style in Swift files (`if`, `switch`, `Task`, closures).
- Prefer line breaks for long initializers over horizontal compression.
- Group related logic with `// MARK:` sections in larger files.

### Types and Declarations
- `struct` for SwiftUI views, value types, and mirrors of NDI C structs.
- `class` (usually `final`) for `ObservableObject`, `NS*` subclasses,
  `DisplayStreamer`, and `NDISender`.
- `enum` for modes, state, and preference keys.
- Use `private`/`fileprivate` aggressively.
- Maintain `@MainActor` isolation for UI/app orchestration types.
- When crossing async/background work back to UI, hop to the main actor
  explicitly (`Task { @MainActor in ... }` or `MainActor.run { ... }`).

### Naming Conventions
- Types: `UpperCamelCase` (`DisplayStreamer`, `StreamingController`).
- Properties/functions/locals: `lowerCamelCase`.
- Boolean names as predicates (`isStreaming`, `ndiReady`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [R44VC0RP/ndi-bar](https://github.com/R44VC0RP/ndi-bar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
