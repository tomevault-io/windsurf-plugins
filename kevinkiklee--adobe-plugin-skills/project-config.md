---
trigger: always_on
description: Use when building, modifying, or debugging Adobe Photoshop UXP plugins or Lightroom Classic plugins — panel UI, pixel access, develop-module observation, memory management in long-lived LrC processes, hybrid native bridges, UXP HTML/CSS/Canvas limits, and Spectrum widgets
---


# Adobe Plugin Development

Reference for writing Photoshop UXP plugins (JavaScript/HTML) and Lightroom Classic plugins (Lua). Covers what the SDKs allow, what they don't, and the non-obvious pitfalls that burn hours.

Baseline: UXP 9.2.0 / Manifest v5 / Photoshop 27.4. LrC 15.3 SDK / Lua 5.1.5.

## When to Use

Use when:
- Writing or editing a UXP panel/command plugin (`manifest.json`, `entrypoints.setup`, Spectrum `sp-*` widgets)
- Writing or editing a Lightroom Classic plugin (`Info.lua`, `LrView`, `LrDevelopController`)
- Reading pixel data from Photoshop (`imaging.getPixels`)
- Wiring develop-slider observation in LrC
- Displaying generated images inside a UXP panel (no HTML canvas pixel ops!)
- Bridging LrC to a native binary for pixel processing
- Debugging memory leaks in long-lived LrC floating dialogs
- Using batchPlay for Photoshop action descriptors
- Manipulating Document/Layer DOM (create, modify, filter, composite)
- Recording or replaying Photoshop actions programmatically

Don't use for:
- General web UI — UXP is NOT a browser; different rules apply
- ExtendScript / CEP (legacy Adobe plugin systems) — this covers UXP only
- Lightroom Cloud REST APIs — those are separate from the LrC desktop SDK

## Core Capability Matrix

| Capability | Photoshop UXP | Lightroom Classic |
|---|---|---|
| Language | JavaScript (HTML/CSS) | Lua 5.1.5 |
| Pixel read | `imaging.getPixels()` | **None** — must export rendition + decode externally |
| Pixel write | `imaging.putPixels()` | None |
| UI | HTML/CSS/JS + Spectrum `sp-*` | `LrView` declarative widgets only |
| Canvas/drawing | Limited 2D (no `drawImage`/`getImageData`/`toDataURL`) | None |
| Document DOM | Full: 33 Document props + 29 methods, 28 Layer props + 55 methods (38 filter + 17 other) | Catalog-centric: `LrCatalog`, `LrPhoto` |
| Selection API | Full `Selection` class (v25.0+) | `LrSelection` namespace (rating, flag, label, nav) |
| AI features | `generativeUpscale` (v27.2+) | denoise, reflection removal, distraction detection |
| Develop events | `action.addNotificationListener` | `LrDevelopController.addAdjustmentChangeObserver` |
| Plugin types | Panel, Command | Export, Publish, Metadata, Filter, Library menu |
| Develop-panel UI | Panels dock anywhere | **Cannot** add Develop-right-panel UI — only floating dialog |
| Action recording | `recordAction` API (v25.0+) | Not available |

**Fundamental constraint:** LrC cannot read pixels from Lua. Any LrC plugin needing pixel access must export a thumbnail or rendition and hand it to a compiled external binary (C++/Rust).

## Critical Pitfalls

### UXP: `<canvas>` has no pixel ops
- `<canvas>` exists (v7.0.0+) but has **no** `drawImage`, `getImageData`, `putImageData`, `toDataURL`, `toBlob`.
- To display a generated image: build pixel buffer -> `imaging.createImageDataFromBuffer` -> `imaging.encodeImageData({ base64: true })` -> set `<img src="data:image/jpeg;base64,...">`.
- Windows extra sharp edges: `createLinearGradient`, `createRadialGradient`, `clearRect` may fail entirely.
- Real-render options: software renderer in JS, embedded WebView, or hybrid C++ plugin.

### UXP: Imaging API gotchas
- 16-bit range is **0-32768 by default**, not 0-65535. Pass `{ fullRange: true }` to `getData()` for full range.
- **Always call `imageData.dispose()` after `getData()`**. UDT warns at 600MB.
- Pass `targetSize` to leverage Photoshop's pyramid cache — dramatically faster than full-res reads.
- `encodeImageData` only supports **RGB** (not Lab, not Grayscale).
- `getPixels()` usually works without `executeAsModal`, but any pixel *write* requires it.
- Default layout is "chunky" (interleaved `RGBRGB`); planar is `RRGGBB`.

### UXP: Layout and CSS limits
- No CSS Grid, no `float`, no `transition`/`@keyframes`, no `text-transform`, no `font` shorthand, no `position: sticky`.
- `box-shadow`, `transform-origin`, `scaleX`/`scaleY`, `translate` **require** the `CSSNextSupport` feature flag in manifest. Without it, CSS silently ignores these properties.
- `window.devicePixelRatio` always returns 1 — cannot detect HiDPI.
- `hide` panel lifecycle callback never fires; `show` fires only once (PS-57284). Don't rely on them for refresh gating.
- `<label for="id">` doesn't work — wrap the label around the control instead.
- `<option>` needs an explicit `value` attribute.

### UXP: CSSNextSupport feature flag
Enable in manifest.json to unlock additional CSS properties:
```json
{
  "featureFlags": { "CSSNextSupport": true }
}
```
Unlocks: `box-shadow`, `transform-origin`, `scaleX`, `scaleY`, `translate`. Without this flag these properties are silently ignored — no error, no warning.

### UXP: localStorage and sessionStorage
Both are available as global storage APIs — contrary to some older references listing them as unavailable. However, they are **not** available inside WebView when loading local content.

### UXP: WebView availability

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kevinkiklee/adobe-plugin-skills](https://github.com/kevinkiklee/adobe-plugin-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
