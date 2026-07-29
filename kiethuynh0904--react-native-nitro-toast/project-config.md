---
trigger: always_on
description: A lightweight, native-powered toast notification library for React Native built with **Nitro Modules** (SwiftUI on iOS, Jetpack Compose on Android). Zero bridge overhead — JSI-native via the Nitro abstraction layer.
---

# react-native-nitro-toast

A lightweight, native-powered toast notification library for React Native built with **Nitro Modules** (SwiftUI on iOS, Jetpack Compose on Android). Zero bridge overhead — JSI-native via the Nitro abstraction layer.

**npm:** `react-native-nitro-toast` · **version:** 1.3.1
**Repo:** `kiethuynh0904/react-native-nitro-toast` · **default branch:** `master`

---

## Architecture

The library is split into three layers that must stay in sync:

```
src/specs/NitroToast.nitro.ts   ← TypeScript interface (source of truth)
        ↓  bun run specs
nitrogen/                        ← auto-generated bridge (DO NOT edit)
        ↓
ios/NativeIntegration/HybridNitroToast.swift    ← iOS entry point
android/.../HybridNitroToast.kt                 ← Android entry point
```

### TypeScript layer (`src/`)

| File | Role |
|------|------|
| `src/specs/NitroToast.nitro.ts` | Nitro `HybridObject` interface — defines `show()` and `dismiss()` |
| `src/index.ts` | Public API: `showToast`, `dismissToast`, `showToastPromise`, `defaultToastConfig` |
| `src/types.ts` | Additional types: `ToastPromiseMessages`, `ToastPromiseConfig` |

Public API is instantiated via:
```ts
const NitroToastModule = NitroModules.createHybridObject<NitroToast>('NitroToast')
```

### iOS layer (`ios/`)

| File | Role |
|------|------|
| `NativeIntegration/HybridNitroToast.swift` | Nitro entry point — dispatches to `ToastViewModel` on main thread |
| `ViewModels/ToastViewModel.swift` | `@MainActor` singleton — owns `UIWindow` and `[Toast]` state |
| `NativeIntegration/PassthroughView.swift` | Custom `UIWindow` subclass that passes touches through |
| `Views/ToastListView.swift` | Alert-mode SwiftUI view |
| `Views/ToastStackView.swift` | Stacked-mode SwiftUI view |
| `Views/ToastIconView.swift` | Icon renderer (predefined types + custom `iconUri`) |
| `Model/Toast.swift` | `Toast` model (ObservableObject) |
| `Helpers/Color+Extensions.swift` | Hex string → `Color` conversion |
| `Assets.xcassets/` | Named color assets for each toast type |

**iOS runtime flow:**
1. `HybridNitroToast.show()` receives call from JS thread
2. Dispatches to `DispatchQueue.main` → `ToastViewModel.shared.present()`
3. If no window exists: creates `PassthroughWindow` at `windowLevel: .alert + 1`, hosts a SwiftUI view
4. `ToastViewModel.emit()` appends to `@Published var toasts`, starts an async countdown task
5. On dismiss: removes from array, tears down `UIWindow` when list is empty

### Android layer (`android/src/main/java/com/margelo/nitro/nitrotoast/`)

| File | Role |
|------|------|
| `HybridNitroToast.kt` | Nitro entry point — delegates to `ToastManager` |
| `ToastManager.kt` | `object` singleton — manages `ComposeView` on `decorView`, coroutine lifecycle |
| `ToastListState.kt` (inside ToastManager.kt) | `MutableStateFlow`-based state for the toast list |
| `ToastList.kt` | Compose rendering (alert + stacked modes) |
| `Toast.kt` | Data class |
| `ToastIcon.kt` | Icon Compose component |
| `DraggableToast.kt` | Swipe-to-dismiss gesture wrapper |
| `Utils.kt` | Hex → `Color`, other utilities |

**Android runtime flow:**
1. `HybridNitroToast.show()` receives call on Nitro thread
2. Delegates to `ToastManager.show()` with the Activity from `NitroModules.applicationContext`
3. `ensureToastContainer()` adds a `ComposeView` to `window.decorView` if not present
4. Coroutine launches `handleToastLifecycle()`: sets visible after 16ms frame delay, auto-dismisses after duration
5. On dismiss: `removeWithAnimation()` hides then removes; container detached when list empty

### Nitro codegen (`nitrogen/`, `nitro.json`)

`nitro.json` configures the Nitro build:
- `cxxNamespace`: `["nitrotoast"]`
- iOS module name: `NitroToast` (autolinking: `HybridNitroToast` Swift class)
- Android namespace: `nitrotoast`, CXX lib: `NitroToast` (autolinking: `HybridNitroToast` Kotlin class)

`nitrogen/` contains auto-generated C++ bridge, Swift specs, and Kotlin specs.
**Never edit files in `nitrogen/` manually.**

---

## Key Commands

```bash
bun run typecheck          # TypeScript type check (no emit)
bun run lint               # ESLint with auto-fix
bun run specs              # tsc + nitro-codegen (regenerate nitrogen/)
bun run prepare            # react-native-builder-bob build → lib/
bun run format:ios         # swiftformat ios/
bun run format:android     # ktlint -F android/src/**/*.kt
bun run release            # release-it (version bump + CHANGELOG + npm publish)
```

**Never edit `lib/` directly** — it is build output from `bun run prepare`.

---

## Nitro Interface Change Workflow

When changing the public native API (adding/removing methods or types in `NitroToast.nitro.ts`):

1. Edit `src/specs/NitroToast.nitro.ts`
2. Run `bun run specs` — this regenerates `nitrogen/`
3. Update `ios/NativeIntegration/HybridNitroToast.swift` to implement new methods
4. Update `android/.../HybridNitroToast.kt` to implement new methods
5. Both platforms must compile before opening a PR

If only changing TypeScript helpers (`src/index.ts`, `src/types.ts`), no codegen is needed.

---

## Release Process


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kiethuynh0904/react-native-nitro-toast](https://github.com/kiethuynh0904/react-native-nitro-toast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
