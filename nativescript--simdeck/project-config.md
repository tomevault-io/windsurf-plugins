---
trigger: always_on
description: This repository is a local-first simulator control plane. The product goal is a native CLI server that can manage iOS simulators, expose an HTTP API, and serve a React web client that shows live simulator output in the browser.
---

# SimDeck Agents Guide

This repository is a local-first simulator control plane. The product goal is a native CLI server that can manage iOS simulators, expose an HTTP API, and serve a React web client that shows live simulator output in the browser.

## Product Shape

- `packages/server/native/` is the native boundary.
- `packages/client/` is the browser UI.
- `skills/simdeck/SKILL.md` is the operator guide for using the tool from Codex.
- `scripts/` holds repeatable build entrypoints.
- `docs/` is the public VitePress documentation site (deployed to GitHub Pages by `.github/workflows/docs.yml`).

The native side should own anything that depends on macOS frameworks, `xcrun simctl`, or private CoreSimulator/SimulatorKit APIs. The web client should stay thin and consume the CLI API.

## Current Architecture

- `packages/server/src/main.rs`
  Owns the CLI entrypoint, Rust subcommands, HTTP server, and static asset serving.
- `packages/server/src/api/routes.rs`
  Defines REST routes for simulator control, health, metrics, and chrome assets.
- `packages/server/src/transport/webrtc.rs`
  Exposes the H.264 WebRTC offer/answer endpoint for browser live video.
- `packages/server/src/webkit.rs`
  Discovers simulator WebKit Remote Inspector targets and bridges WebInspectorUI
  WebSocket traffic to the simulator `webinspectord` binary-plist socket.
- `packages/server/src/simulators/registry.rs`
  Tracks Rust-side simulator session state and lazy native attachment by UDID.
- `packages/server/native/XCWSimctl.*`
  Wraps `xcrun simctl` for discovery, lifecycle management, app launching, URL opening, appearance toggles, and simulator log capture.
- `packages/server/native/DFPrivateSimulatorDisplayBridge.*`
  Owns headless private display frames plus HID-based touch and keyboard injection.
- `packages/server/native/XCWAccessibilityBridge.*`
  Owns private CoreSimulator accessibility snapshots through `AccessibilityPlatformTranslation`.
- `packages/server/native/XCWPrivateSimulatorSession.*`
  Owns one private display bridge per booted simulator plus selectable hardware/software H.264 encode.
- `packages/server/native/bridge/XCWNativeBridge.*`
  Narrow C ABI for simulator control, chrome rendering, and native frame callbacks into Rust.
- `packages/server/native/bridge/XCWNativeSession.*`
  Wraps one Objective-C private simulator session handle for the Rust registry.
- `packages/server/native/XCWPrivateSimulatorBooter.*`
  Uses private `CoreSimulator` APIs for direct simulator boot without launching Simulator.app.
- `packages/server/native/XCWChromeRenderer.*`
  Renders Apple’s CoreSimulator device-type PDF chrome assets into PNGs for the browser.
- `packages/client/src/app/App.tsx`
  Browser entrypoint for the React control surface.
- `packages/nativescript-inspector/src/index.ts`
  NativeScript in-app inspector runtime that connects to the Rust server over
  WebSocket, publishes NativeScript/UIKit hierarchies, and performs debug UIKit
  property edits from JavaScript.
- `packages/react-native-inspector/src/index.ts`
  React Native in-app inspector runtime that connects to the Rust server over
  WebSocket, publishes React Fiber component hierarchies with Metro source
  locations, and performs best-effort debug JS/native prop edits.
- `packages/flutter-inspector/lib/simdeck_flutter_inspector.dart`
  Flutter in-app inspector runtime that connects to the Rust server over
  WebSocket, publishes widget/render/semantics hierarchies with debug creation
  locations, and performs best-effort semantics, focus, text, and scroll actions.

## Working Rules

- Keep simulator-native logic in Objective-C under `packages/server/native/`.
- Keep Rust server logic under `packages/server/`.
- Keep browser-only presentation logic in `packages/client/`.
- Keep NativeScript app runtime inspection logic in `packages/nativescript-inspector/`.
- Keep React Native app runtime inspection logic in `packages/react-native-inspector/`.
- Keep Flutter app runtime inspection logic in `packages/flutter-inspector/`.
- Prefer adding a native API endpoint before adding client-only assumptions.
- Do not add a Node or Swift dependency to solve work that already fits in Foundation/AppKit.
- When touching private API usage, keep the adaptation small and explicit and document any simulator/runtime assumptions here.
- Prefer stable CLI subcommands over hidden environment variables.

## Private API Notes

Private simulator behavior is implemented locally in:

- Boot path: `packages/server/native/XCWPrivateSimulatorBooter.*`
- Full live display bridge: `packages/server/native/DFPrivateSimulatorDisplayBridge.*`
- Accessibility bridge: `packages/server/native/XCWAccessibilityBridge.*`

The current repo uses the private boot path, private display bridge, and private accessibility translation bridge directly. The browser streams frames from that bridge, injects touch and keyboard events through the same native session layer, inspects accessibility through `AccessibilityPlatformTranslation`, and renders device chrome from `packages/server/native/XCWChromeRenderer.*`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NativeScript/SimDeck](https://github.com/NativeScript/SimDeck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
