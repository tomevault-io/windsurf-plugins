---
trigger: always_on
description: Things that cost time to find out and cannot be read off the source. Users
---

# AGENTS.md

Things that cost time to find out and cannot be read off the source. Users
want `README.md`; this is the traps, the outside contracts, and how to run it.

OmaSnap is an Omarchy shell plugin (`tahayvr.omasnap`) living inside the
long-lived Quickshell process `omarchy-shell`: an `overlay` (the editor) and a
`bar-widget`, `keepLoaded` so the last edit survives a hide. Capture, OCR,
image encoding and the clipboard are delegated to `omarchy`, `tesseract`,
ImageMagick and `wl-copy`; the plugin owns the beautify, annotate and export
stage only.

## Shell contract

Imposed by the host, so none of it is negotiable from in here.

- The host injects `omarchyPath`, `shell` and `manifest` into the overlay root
  *after* load. `shell` is a capability-scoped facade that can only act on this
  plugin's own id.
- `summon <id> <json>` calls `open(payloadJson)`, `hide <id>` calls `close()`.
  **`close()` must be idempotent**: `dismiss()` calls it and then
  `shell.hide()`, which calls it again. Never close from the inside without
  telling the shell, or `toggle` desyncs.
- `call <id> <fn> <arg>` invokes any root function and returns its string
  result (`undefined` becomes `ok`). Public surface: `edit`, `capture`, `code`,
  `pick`, `save`, `saveAs`, `copy`, `crop`, `uncrop`, `redact`, `copyText`,
  `set`, `info`, `annotate`.
  Keep those names stable; the README documents them. `info` cannot be called
  `state` because `Item` already has one.
- **The CLI splits an argument starting with `[` on commas, and splits on
  literal spaces.** So `annotate` takes `{"items": [...]}` rather than a bare
  array, and any string value needs a literal \u0020 instead of a space.
- Bar widgets get `bar`, `moduleName`, `settings`. `bar.shell` is the same
  facade the overlay gets, wired per entry by the shell's own `Bar.qml`.
- Payloads: `{"path":…}`, `{"capture":"region|windows|fullscreen|smart"}`,
  `{"code":true}`, `{"text":…}`, and `{}` for the empty starting state.
  Styling survives between opens; content does not.

## Rendering

`Stage` is what `grabToImage()` grabs. It is laid out in shot pixels times
`unit = 1 / dpr` and displayed scaled by `viewport.fit`.

- **`grabToImage(cb, size)` multiplies `size` by the window's *effective*
  DPR** — 1.6 on a fractionally scaled monitor, while `Screen.devicePixelRatio`
  claims 2 — and the stage is then not a whole number of logical pixels
  (1210 shot pixels are 756.25). Grabbing it at 757 renders at 757/756.25 and
  resamples everything, and a padding-0 export came out 1101 wide. So the
  export grabs `grabRoot`, a wrapper padded up to whole device pixels
  (`Model.grabSize`), and `snap-deliver` crops the surplus. `Stage.dpr` reads
  `Window.window.devicePixelRatio`.
- **Never draw the shot through a texture round trip.** `ClippingRectangle`,
  `layer.enabled`, `layer.textureSize` and a `MultiEffect` mask all resample
  it on a fractional scale — measured under Quickshell: 8% of the card's
  pixels changed, some by the full 255, while the plain `qml` runtime and the
  test stub came out exact, which is why the harness never saw it. The shot
  is a `Shape` whose `ShapePath.fillItem` is the hidden `Image`: that samples
  the image's own texture one texel per shot pixel, rounded corners included.
- **The `unit` scaling is load-bearing.** The stage is laid out in shot
  pixels times `unit`, so a stage scale of 1 shows the shot life-size and
  the grab is 1:1. `AnnotationLayer` and `CodeBlock` keep shot-pixel
  coordinates and are placed with `scale: unit`; `Editor.toShot` inverts it.
- **The shadow is a `ShaderEffect` too** (`ui/Shadow.qml`,
  `assets/shaders/shadow.frag`): the closed-form Gaussian blur of a rounded
  box, dithered. `MultiEffect`'s shadow stepped into rings at the radii a
  padded frame asks for and broke down past `blurMax` 64, and it needed the
  cards drawn over a hidden copy of themselves. Nothing goes through
  `MultiEffect` now except the wordmark's colorisation.
- **`doc.exporting`** is raised for the grab frame. Anything that must not
  reach the file — selection outlines, the empty-text placeholder — binds to it.
- **Linear ramps are a `ShaderEffect`** (`ui/Ramp.qml`,
  `assets/shaders/ramp.frag`), not a `Rectangle` gradient: an 8-bit ramp
  steps a level every dozen pixels and a dark auto background showed the
  bands. Only noise added *before* quantisation cures that; grain laid over
  the finished gradient leaves the step in the local mean and only hides it
  once it is visible itself (tried at 1% and 4.5%, both rejected). Rebuild
  the `.qsb` with the command in the shader's header after editing it, and
  commit both. Meshes still go through `QtQuick.Shapes`. `half` is a
  reserved word in the shader language qsb compiles.
- **The spotlight dim is one filled path for every spotlight**
  (`ui/Spotlight.qml`, `Model.spotlightPath`): drawn one per annotation they
  would darken twice where two overlap. It is the picture's outline plus a
  subpath per spotlight, filled odd-even, and a hole is clamped to the
  outline — past it the odd-even rule fills the hole in instead of punching
  it out, which showed as a dark wedge outside the card. A `ListModel` emits
  nothing a binding can follow, so the path rides on `doc.annotationRevision`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tahayvr/postcard](https://github.com/tahayvr/postcard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
