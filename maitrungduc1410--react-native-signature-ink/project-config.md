---
trigger: always_on
description: Operational guide for AI coding agents (and new contributors) working on this repo.
---

# AGENTS.md

Operational guide for AI coding agents (and new contributors) working on this repo.

**Read in this order:**

1. This file — operational quick-reference.
2. [`LESSONS_LEARNED.md`](LESSONS_LEARNED.md) — the narrative behind every gotcha listed below. Several bugs in this repo were "fixed" three or four times before we understood the actual rule; that file is the difference between you re-fixing them again or not. **Read it before you touch view recycling, the tool picker, Android layout, or unit conversion.**
3. [`ARCHITECTURE.md`](ARCHITECTURE.md) — how the library is structured end-to-end.
4. [`README.md`](README.md) — public docs.
5. [`CONTRIBUTING.md`](CONTRIBUTING.md) — workflow conventions.

## Project overview

`react-native-signature-ink` is a fully native React Native signature library. iOS is Swift on top of `PKCanvasView` (PencilKit); Android is Kotlin running a hand-tuned velocity-Bezier ink algorithm into an offscreen `Bitmap`. The JS surface is a Fabric codegen component plus one ergonomic wrapper with a Promise-based imperative API. No Skia, no Reanimated, no WebView, no native modules — view-only library.

The repo is a Yarn monorepo: the library lives at the root, the demo app at `example/`.

## Setup

```sh
# Install workspace deps (root)
yarn

# Run the example app
yarn example ios          # builds + runs on the iOS simulator
yarn example android      # builds + runs on a device/emulator
yarn example start        # Metro only

# Static checks
yarn typecheck            # tsc against the workspace
yarn lint                 # eslint
```

Native changes require a full rebuild (`yarn example ios|android`). JS-only changes hot-reload through Metro.

The example app's Metro config uses `react-native-monorepo-config`. If you bump dependencies and `metro.config.js` starts failing, double-check the example's `package.json` resolutions block — Metro is the most fragile piece of the workspace setup.

## Code style

### TypeScript

- Strict mode. JSDoc on every public type, prop, and method. Defaults documented inline.
- Two entry shapes: the high-level wrapper (`SignatureInk`, in [`src/SignatureInk.tsx`](src/SignatureInk.tsx)) is the recommended surface; the raw codegen component (`SignatureInkView`, in [`src/SignatureInkViewNativeComponent.ts`](src/SignatureInkViewNativeComponent.ts)) is the escape hatch.
- The codegen file is the **single source of truth** for native props, commands, and event payload shapes. Both platforms generate their Fabric glue from it.

### Swift (iOS)

- Every prop the Obj-C++ host reads must be declared `@objc public var` in [`ios/SignatureInkSurface.swift`](ios/SignatureInkSurface.swift). Use `didSet` to fan out side effects (rebuild toolbar, sync tool picker, etc.).
- **Every** `@objc public var` MUST be reset to its declared default inside `prepareForReuse`. The list of resets must stay in sync with the list of declarations — see the "View recycling" gotcha below.
- Keep PencilKit types out of the Obj-C++ wrapper ([`ios/SignatureInkView.mm`](ios/SignatureInkView.mm)) — PencilKit isn't imported there. The split between the Obj-C++ wrapper and the Swift surface exists precisely for this reason.

### Kotlin (Android)

- All user-facing dimensions (pen widths, baseline width, `baselineOffsetFromBottom`, toolbar height / spacing) are stored in **dp** internally and converted to raw pixels at every draw site via the `dpToPx` helper. This is non-negotiable — see the "Pen widths" gotcha below.
- Layout for the `SignatureInkView` parent is performed **synchronously** in setters via `applyChildLayout()`. Do not add a setter that mutates layout state without calling it.
- The renderer view ([`SignatureCanvasView.kt`](android/src/main/java/com/signatureink/SignatureCanvasView.kt)) draws into an offscreen `inkBitmap`. Every export (PNG / JPEG / SVG / clipboard / photo library) reads from that bitmap, so exports are instant.

### Comments

- Block size: **≤ ~10 lines** unless the rationale is genuinely irreducible.
- Capture **why** the code is the way it is — what breaks if you change it, what's load-bearing, what's non-obvious. Don't restate what the code does.
- Lean on the JSDoc in [`src/types.ts`](src/types.ts) / [`src/SignatureInkViewNativeComponent.ts`](src/SignatureInkViewNativeComponent.ts); don't duplicate.

## Architecture pointers

| Concern | File |
| --- | --- |
| Public JS API + Promise plumbing | [`src/SignatureInk.tsx`](src/SignatureInk.tsx) |
| Public types | [`src/types.ts`](src/types.ts) |
| Codegen spec (source of truth) | [`src/SignatureInkViewNativeComponent.ts`](src/SignatureInkViewNativeComponent.ts) |
| iOS Fabric host | [`ios/SignatureInkView.mm`](ios/SignatureInkView.mm) |
| iOS rendering surface | [`ios/SignatureInkSurface.swift`](ios/SignatureInkSurface.swift) |
| Android Fabric host + layout | [`android/src/main/java/com/signatureink/SignatureInkView.kt`](android/src/main/java/com/signatureink/SignatureInkView.kt) |
| Android renderer | [`android/src/main/java/com/signatureink/SignatureCanvasView.kt`](android/src/main/java/com/signatureink/SignatureCanvasView.kt) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maitrungduc1410/react-native-signature-ink](https://github.com/maitrungduc1410/react-native-signature-ink) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
