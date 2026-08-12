---
trigger: always_on
description: Guidelines for AI agents (Claude, Codex, etc.) working on this codebase. Read this before making any changes.
---

# AGENTS.md — Vue Native

Guidelines for AI agents (Claude, Codex, etc.) working on this codebase. Read this before making any changes.

---

## What this project is

Vue Native is a framework for building **real native iOS, Android, and macOS apps with Vue 3**. It is NOT a WebView wrapper. Vue components drive `UIKit` views on iOS, `Android Views` on Android, and `AppKit` views on macOS via a custom `createRenderer()` bridge.

```
Vue SFC  →  Vue custom renderer  →  NativeBridge (TS)
                                          ↓  JSON batch
                                  iOS: Swift → UIKit + Yoga
                                  Android: Kotlin → Views + FlexboxLayout
                                  macOS: Swift → AppKit + LayoutNode
```

---

## Repository layout

```
packages/
  runtime/          @thelacanians/vue-native-runtime — renderer, bridge, components, composables
  navigation/       @thelacanians/vue-native-navigation — createRouter, RouterView, useRouter
  vite-plugin/      @thelacanians/vue-native-vite-plugin — Vite build config for native targets
  cli/              @thelacanians/vue-native-cli — project scaffold + dev tooling
native/
  shared/VueNativeShared/    Cross-platform Swift (iOS 16+, macOS 13+)
    Sources/VueNativeShared/
      JSRuntime, EventThrottle, HotReloadManager, CertificatePinning,
      SharedJSPolyfills, NativeModule protocol, NativeModuleRegistry,
      NativeEventDispatcher, Modules/ (9 shared modules)
    Tests/VueNativeSharedTests/
  ios/VueNativeCore/         Swift Package (iOS 16+, UIKit, JavaScriptCore, Yoga)
    Package.swift
    Sources/VueNativeCore/
      Bridge/                JSRuntime, NativeBridge, VueNativeViewController,
                             HotReloadManager, JSPolyfills, ErrorOverlayView,
                             EventThrottle, CertificatePinning
      Components/Factories/  One Swift factory per component (VButtonFactory, etc.)
      Modules/               Native module implementations (Haptics, Camera, etc.)
      Styling/               StyleEngine.swift — CSS props → UIKit/Yoga
      Helpers/               Extensions, GestureWrapper, TouchableView, UIColor+Hex
    Tests/VueNativeCoreTests/
  android/VueNativeCore/     Android library (API 21+, Kotlin, J2V8/V8, FlexboxLayout)
    src/main/kotlin/com/vuenative/core/
      Bridge/                JSRuntime, NativeBridge, HotReloadManager, JSPolyfills,
                             ErrorOverlayView
      Components/Factories/  One Kotlin factory per component
      Modules/               Native module implementations
      Styling/               StyleEngine.kt — CSS props → FlexboxLayout/View
      Helpers/               EventThrottle, GestureHelper, TouchableView
      VueNativeActivity.kt   Base Activity for apps
    src/test/kotlin/com/vuenative/core/
  macos/VueNativeMacOS/      Swift Package (macOS 15+, AppKit, JavaScriptCore, LayoutNode)
    Package.swift             (depends on VueNativeShared)
    Sources/VueNativeMacOS/
      Bridge/                NativeBridge, JSPolyfills, VueNativeWindowController,
                             ErrorOverlayView, HotReloadManager, EventThrottle
      Components/Factories/  28 component factories (AppKit equivalents)
      Modules/               16 modules (12 cross-platform + 4 macOS-only)
      Styling/               StyleEngine.swift — CSS props → LayoutNode + NSView.layer
      Layout/                LayoutNode (custom flexbox), FlippedView (isFlipped=true)
      Helpers/               ClickableView, GestureWrapper, NSColor+Hex, Extensions
    Tests/VueNativeMacOSTests/
docs/
  src/               VuePress 2.x documentation site (bun run dev in docs/)
examples/
  counter/  calculator/  settings/  social/  tasks/  todo/
  auth-flow/  chat/  camera-app/  forms/  lists/
  navigation-demo/  media-player/  theming/  macos-showcase/
tools/
  vscode-extension/  VS Code snippets + diagnostics
  nvim-plugin/       Neovim snippets + completion + diagnostics
```

---

## Progress, changelog, and backlog notes

Deep reviews and long-running uncommitted work need a local handoff trail. Keep that trail outside the repository so it can guide future work without becoming a release artifact:

```bash
~/notes/projects/vue-native/
```

This is the canonical handoff location. `~/notes/vue-native-backlog/` is a
legacy compatibility mirror only; new notes should be organized under the
project-specific directory above.

These notes are for agents and maintainers continuing the same branch. They should answer four questions quickly:

- What changed in the current uncommitted tree?
- What was reviewed and what validation passed?
- What is still risky, blocked, or unverified?
- What should the next agent do first?

### Current local notes

- `~/notes/projects/vue-native/progress-2026-07-10.md`
- `~/notes/projects/vue-native/handoff-2026-07-10.md`
- `~/notes/projects/vue-native/CHANGELOG-draft-2026-07-10.md`
- Historical notes: `~/notes/projects/vue-native/archive/2026-05-26/` and `~/notes/projects/vue-native/archive/2026-07-09/`

### When to create or update notes

Update the notes whenever you do any of the following:

- Perform a broad codebase review or audit.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [abdul-hamid-achik/vue-native](https://github.com/abdul-hamid-achik/vue-native) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
