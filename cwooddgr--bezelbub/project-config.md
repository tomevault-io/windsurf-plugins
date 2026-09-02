---
trigger: always_on
description: macOS + iOS app that composites device bezels onto screenshots and screen recordings. Built with SwiftUI, targeting macOS 14+ and iOS 17+.
---

# Bezelbub

macOS + iOS app that composites device bezels onto screenshots and screen recordings. Built with SwiftUI, targeting macOS 14+ and iOS 17+.

The device-framing engine lives in a UI-free Swift package (`BezelbubKit`) so it can run headless. The SwiftUI apps, the share extension, and the `bezelbub` CLI are all thin clients of that package — see [Architecture](#architecture-shared-engine--thin-adapters).

## Project Structure

- `BezelbubKit/` — **UI-free engine** as a local SwiftPM package (the single source of truth for framing logic and bezel assets)
  - `Sources/BezelbubKit/` — `FrameCompositor` (the pure screenshot→framed-image transform, Core Graphics), `DeviceMatcher`, `DeviceDefinition`/`DeviceCatalog`, `ScreenRegionDetector`. No SwiftUI, no app state, no AVFoundation.
  - `Sources/BezelbubKit/Resources/` — `Bezels/`, `Masks/`, `screen-regions.json`, served to every consumer via `Bundle.module`
  - `Sources/BezelbubVideoKit/` — video framing as a **separate library product** (`VideoFrameCompositor`, `BezelOverlayCompositor`, `BezelOverlayInstruction`; AVFoundation/CoreImage) so still-image consumers like the share extension don't link the video pipeline
  - `Sources/bezelbub/` — the `bezelbub` CLI (swift-argument-parser); `frame` and `devices` subcommands
  - `Tests/BezelbubKitTests/` — engine round-trip tests (image + video)
- `Shared/` — Cross-platform **app** code (compiled into the app targets; imports `BezelbubKit` + `BezelbubVideoKit`)
  - `AppState.swift` — Application state (uses `#if os()` for platform-specific bits; the share extension compiles it with `-DSHARE_EXTENSION`, which guards out the video/`BezelbubVideoKit` parts)
  - `Models/` — App-layer models (`ExportSizeModel`)
- `macOS/` — macOS-specific code
  - `BezelbubApp.swift` — macOS app entry point
  - `Views/` — macOS SwiftUI views (`ContentView`, `ExportSizeAccessoryView`)
  - `Info.plist`, `Bezelbub.entitlements`, `Assets.xcassets`
- `iOS/` — iOS-specific code
  - `BezelbubApp.swift` — iOS app entry point
  - `Views/` — iOS SwiftUI views (`ContentView` with PhotosPicker, share sheet)
  - `Info.plist`, `Bezelbub-iOS.entitlements`, `Assets.xcassets`
- `BezelbubShareExtension/` — iOS Share Extension
  - `ShareViewController.swift` — Receives images from share sheet, frames them, copies to clipboard
  - `Info.plist`, `BezelbubShareExtension.entitlements`
- `Apple Product Bezels/` — Source bezel PNGs from Apple (gitignored, local reference only)
- `project.yml` — XcodeGen project definition

## Architecture: shared engine + thin adapters

The pure transformation `(screenshot, device id, orientation, styling) → framed image` lives in **`BezelbubKit`** and has no dependency on SwiftUI, app state, AVFoundation, or a GUI session — it's Core Graphics bitmap work, so it runs fully offscreen (SSH, launchd, CI). Bezel/mask assets ship inside the package and resolve via `Bundle.module`, so there's one copy regardless of consumer.

Video framing (`(video, device id, orientation, styling) → framed MP4`, audio preserved) lives in the sibling product **`BezelbubVideoKit`** in the same package. It's AVFoundation/CoreImage work: macOS composites via `AVVideoCompositionCoreAnimationTool`, iOS via a custom `AVVideoCompositing` (CALayer alpha doesn't composite correctly on iOS). A `.transparent` background (`VideoBackground` enum) exports HEVC-with-alpha as `.mov` instead of MP4 and routes through the custom compositor on **both** platforms — the CALayer path's alpha behavior is unverified, while the Core Image compositor is deterministic about alpha.

Clients:
- The **macOS / iOS apps** depend on both library products; the **share extension** depends only on `BezelbubKit` (declared in `project.yml` under each target's `dependencies:`). `AppState` and the views are adapters over the engine.
- The **`bezelbub` CLI** (`Sources/bezelbub/`) depends on `BezelbubKit` + `BezelbubVideoKit` + swift-argument-parser. swift-argument-parser is a dependency of the CLI target only — it is **not** linked into the apps (verified: absent from the archived app binary), though it does appear in the package's resolved graph.

A future MCP server is meant to wrap the CLI (a clean process boundary) or link `BezelbubKit` directly.

## Build

The apps are generated with [XcodeGen](https://github.com/yonaskolb/XcodeGen) from `project.yml`:

```sh
xcodegen generate
open Bezelbub.xcodeproj
```

The engine + CLI build with SwiftPM:

```sh
cd BezelbubKit
swift build            # library + CLI
swift test             # engine round-trip tests
swift build --product bezelbub   # just the CLI
```

### CLI usage

```sh
bezelbub devices [--input <path> | --dimensions WxH] [--json]
bezelbub frame --input <path> [--device <id>] [--color <name>] \
               [--orientation portrait|landscape|auto] \
               [--background <hex>|transparent] [--webm] \
               [--output-size <width|WxH|N%>] [--output <path>] [--json]
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cwooddgr/bezelbub](https://github.com/cwooddgr/bezelbub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
