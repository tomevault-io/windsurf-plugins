---
trigger: always_on
description: Guidance for AI coding agents and contributors working in this repo. Keep changes accurate to the native code — most behaviour lives in Swift/Kotlin, not JS. For the full design rationale, read [ARCHITECTURE.md](./ARCHITECTURE.md). Before changing the flight/registry/overlay logic, skim [LESSONS_LEARNED.md](./LESSONS_LEARNED.md) — it records the hard-won bugs and the rules that prevent re-introducing them (and add an entry when you hit a new non-obvious one).
---

# AGENTS.md

Guidance for AI coding agents and contributors working in this repo. Keep changes accurate to the native code — most behaviour lives in Swift/Kotlin, not JS. For the full design rationale, read [ARCHITECTURE.md](./ARCHITECTURE.md). Before changing the flight/registry/overlay logic, skim [LESSONS_LEARNED.md](./LESSONS_LEARNED.md) — it records the hard-won bugs and the rules that prevent re-introducing them (and add an entry when you hit a new non-obvious one).

## What this is

`react-native-shared-hero` is a **Fabric (New Architecture) view component** that performs native shared-element ("hero") transitions. A `<SharedHero id="..." namespace="...">` registers itself with a process-wide native registry on mount and unregisters on unmount; when a matching twin appears or disappears, the registry captures a snapshot and runs a flight in a window-level overlay. It is **router-agnostic** — it never imports a navigation library. The example app uses `@react-navigation/native-stack` + `react-native-screens` only to exercise it.

## Repo layout

- `src/` — the JS/TS public surface.
  - `index.tsx` — exports `SharedHero` (alias of `SharedHeroView`), `useSharedHero`, and the public types.
  - `SharedHeroView.tsx` — the React component; maps friendly props (`id`, `namespace`, `spring`, …) onto the codegen native props (`heroId`, `heroNamespace`, `springDamping`, …) and adapts the native events.
  - `SharedHeroViewNativeComponent.ts` — the Fabric `codegenNativeComponent` spec (`NativeProps`, direct event handlers). This drives codegen.
  - `types.ts` — `SharedHeroProps` and the enums. Source of truth for the public API.
  - `useSharedHero.ts` — JS-only state helper for in-place toggles (no native calls).
- `ios/` — Swift + Obj-C++ implementation (see ARCHITECTURE.md for each file).
  - `SharedHeroView.h/.mm` (Fabric `RCTViewComponentView` shim), `SharedHeroViewImpl.swift` (per-view behaviour + snapshot/geometry), `HeroRegistry.swift` (matching + flight scheduling), `FlightEngine.swift` (the animation), `OverlayHost.swift` (the overlay `UIWindow`), `InteractiveStackPop.swift` + `InteractiveModalReturn.swift` (interactive returns), `SystemZoomBridge.swift` (v2 stub).
- `android/src/main/java/com/sharedhero/` — Kotlin implementation.
  - `SharedHeroView.kt` (the `ReactViewGroup`), `SharedHeroViewManager.kt` (Fabric view manager + style-prop interceptor), `HeroRegistry.kt`, `FlightEngine.kt`, `OverlayHost.kt`, `HeroSnapshot.kt`.
- `example/` — the demo app (Yarn workspace), one screen per use case under `example/src/screens/**`, wired in `example/src/App.tsx` / `navigation.ts`.

## New Architecture / codegen

- The component is Fabric-only. `codegenConfig` in `package.json` declares the spec `SharedHeroViewSpec` with `jsSrcsDir: "src"`, Android package `com.sharedhero`, and the iOS component `SharedHeroView` → class `SharedHeroView`.
- Native prop names are the codegen names (`heroId`, `heroNamespace`, `mode`, `duration`, `springDamping/Stiffness/Mass`, `fadeMode`, `easing`, `motionPath`, `enabled`, `returnFlightEnabled`) plus `onTransitionStart`/`onTransitionEnd` direct events carrying `{ id, ns }`. If you add a prop, update **all** of: `types.ts`, `SharedHeroView.tsx`, `SharedHeroViewNativeComponent.ts`, the iOS `updateProps` in `SharedHeroView.mm` + `SharedHeroConfig`, and the Android `@ReactProp` setter + `SharedHeroConfig`.
- Android style props (`borderRadius`, `overflow`, `borderWidth/Color/Style`) are intercepted by `HeroStylePropDelegate` and routed through `BackgroundStyleApplicator` — codegen-fronted components otherwise drop them. Don't remove that delegate.

## Build / test / lint

Root `package.json` scripts:

- `yarn typecheck` → `tsc` (type-checks the library).
- `yarn lint` → `eslint "**/*.{js,ts,tsx}"`.
- `yarn prepare` → `bob build` (builds `lib/` via react-native-builder-bob; ESM + TypeScript defs).
- `yarn clean` → removes build outputs.

Example app (run from repo root; it's a workspace):

- `yarn example start` — Metro.
- `yarn example ios` / `yarn example android` — build + run.
- Type-check the example separately: `npx tsc --noEmit -p example/tsconfig.json`.

Important: **native changes (Swift/Kotlin) cannot be validated by a JS reload.** You must rebuild through Xcode / Gradle (or `yarn example ios` / `yarn example android`) for `ios/` or `android/` edits to take effect. A Metro Fast Refresh only re-runs JS.

There is no automated test suite; verification is by running the example and watching the flights (the native code logs heavily via `NSLog`/`Log.d` with tags like `SharedHeroRegistry`, `SharedHeroFlight`, `SharedHeroStackPop`).

## Key invariants — do not break


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maitrungduc1410/react-native-shared-hero](https://github.com/maitrungduc1410/react-native-shared-hero) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
