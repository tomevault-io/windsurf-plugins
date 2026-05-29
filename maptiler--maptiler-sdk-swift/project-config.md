---
trigger: always_on
description: <!-- AGENT_DIRECTIVES v1 -->
---

<!-- AGENT_DIRECTIVES v1 -->

Priority: Follow these directives unless they conflict with system/developer instructions or safety rules.

## System Context

You are an AI agent specialized in Swift and JavaScript development. This document defines mandatory rules for consistent, secure, and maintainable development. Follow the sections below precisely:

- Project Overview
- Pre-Implementation Checklist
- Bridge Rules
- Code Style Guidelines
- SwiftLint Compliance
- Development Best Practices
- Swift Concurrency
- Project Structure
- Style Lifecycle
- Error Handling
- Privacy
- Tests
- Pull Request Template

## Project Overview

This Swift Package wraps the MapTiler JS SDK via a typed Swift↔JS bridge located in `Sources/MapTilerSDK/Bridge`. It renders a web map in `MTMapView` (WKWebView) using `Resources/MapTilerMap.html` which loads `maptiler-sdk.umd.min.js`. Swift APIs translate to JS via `MTCommand`, executed by the bridge. The JS SDK API reference is in `js/docs` (browse `index.html` or search with `rg` or `grep`). Always consult the docs before wrapping any function.`


### Main Components

- UI: `MTMapView` hosts a WKWebView rendering `Resources/MapTilerMap.html`.
- Bridge: Public Swift APIs call `MTCommand`s serialized to JS, executed by `WebViewExecutor`; results decode through `MTBridgeReturnType`.
- Lifecycle: `EventProcessor` listens to JS events, updates `MTMapView` state (`style`, `isInitialized`) and notifies delegates.
- Safety: Mutate map after `didLoad`/`isReady`. Style changes reset layers; `MTStyle` handles queueing.
- Responsibilities: `MTBridge` executes commands; `MTCommand` defines JS to run; `WebViewExecutor` calls `evaluateJavaScript` on WKWebView.

## Pre-Implementation Checklist
  Before writing ANY new code, you MUST:
  - Search for existing related types: `Grep pattern="MT[TypeName]|[RelatedConcept]"`.
  - Read similar existing implementations completely.
  - Identify the established patterns and types used.
  - Confirm no existing types can be reused before creating new ones.
  - Follow SwiftLint rules defined in `.swiftlint.yml` (4-space indentation, trailing newlines, etc.).
  - Run `swiftlint lint --quiet` and fix all violations before concluding work. Skip if swiftlint is not available.
  - Don't proceed until all search/pattern analysis is complete.


## Bridge Rules

MUST follow this end-to-end flow when wrapping a JS API into Swift:

1) Discover and design
- Read the JS API in MapTiler SDK for JS docs; determine parameters, defaults, and return type.
- Define a Swift `struct` conforming to `MTCommand` with strongly typed, Codable parameters.

2) Encode parameters
- Use existing Codable helpers to build a compact JSON payload; avoid manual string concat where possible.
- Validate/clamp numeric ranges in Swift prior to execution (zoom, pitch, bearing, durations).

3) Implement `toJS()`
- Build a `JSString` that calls the JS API. Prefer passing a single options JSON object.
- If easing functions or callbacks are needed, convert to a JS expression string (see existing commands for `easing.toJS()`).

4) Choose execution path and return type
- For commands with no meaningful return: `runCommand(_:)`.
- For numeric result: `runCommandWithDoubleReturnValue(_:)`.
- For boolean: `runCommandWithBoolReturnValue(_:)`.
- For string: `runCommandWithStringReturnValue(_:)`.
- For coordinates: `runCommandWithCoordinateReturnValue(_:)`.
- If a new return type is required, extend `MTBridgeReturnType` in a focused change.

5) Public API surface
- Add a thin convenience method on `MTMapView` (or a relevant extension) that:
  - Ensures the map/style are ready (`didLoad`/`isReady`).
  - Validates inputs and applies sensible defaults.
  - Calls the bridge using the appropriate `runCommand*` helper.
  - Surfaces a completion with `Result<…, MTError>` or `async` variant.

6) Threading and lifecycle
- MUST run on the main thread when touching `MTMapView` or UIKit.
- Avoid firing commands before the WebView/bridge is available; prefer queuing until ready.

7) Testing
- Unit test: parameter encoding and range clamping.
- Contract test: `toJS()` shape for simple cases (e.g., duration only).

Example skeleton:

```swift
package struct RotateTo: MTCommand {
    var bearing: Double
    var durationMs: Double?

    package func toJS() -> JSString {
        struct Options: Codable { let bearing: Double; let duration: Double? }
        let opts = Options(bearing: bearing, duration: durationMs)
        let json = opts.toJSON() ?? "{}"
        return "\(MTBridge.mapObject).rotateTo(\(json));"
    }
}

public extension MTMapView {
    func setBearing(_ bearing: Double, durationMs: Double? = nil, completion: ((Result<Void, MTError>) -> Void)? = nil) {
        let clamped = (bearing.truncatingRemainder(dividingBy: 360) + 360).truncatingRemainder(dividingBy: 360)
        runCommand(RotateTo(bearing: clamped, durationMs: durationMs), completion: completion)
    }
}
```
### Add a new command (checklist)

- Read target JS API in `js/docs` and confirm params/return.
- Create `struct` + internal `Options: Codable` (if needed).
- Implement `toJS()` with a single options object.
- Choose correct `runCommand*` by return type.
- Add `MTMapView` convenience API; validate readiness and inputs.
- Tests: encoding, clamping, and `toJS()` contract.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maptiler/maptiler-sdk-swift](https://github.com/maptiler/maptiler-sdk-swift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
