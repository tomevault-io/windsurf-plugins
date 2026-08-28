---
trigger: always_on
description: valo is a 2D render engine in Rust on wgpu. It records drawing commands into a display list, plans them into GPU passes, and draws them alongside a Skia-shaped text stack. The architecture follows Flutter's Impeller; the text stack follows Skia. The host application owns all IO and presentation — windows, files, the network — and the renderer holds no application content of its own, only caches.
---

# valo — how this project works

valo is a 2D render engine in Rust on wgpu. It records drawing commands into a display list, plans them into GPU passes, and draws them alongside a Skia-shaped text stack. The architecture follows Flutter's Impeller; the text stack follows Skia. The host application owns all IO and presentation — windows, files, the network — and the renderer holds no application content of its own, only caches.

## The frame

```
record    DisplayListBuilder → Arc<DisplayList>      CPU only, no GPU device, any thread
plan      cull → depth assign → reorder → segment    pure CPU pass over the recorded ops
encode    render passes, breaking only where recorded (backdrop filters, advanced blends)
submit    stats: cpu/plan/encode ms, draws, culled, passes, atlas churn, GPU timestamps
```

## Crates

| crate | role |
|---|---|
| `valo-geometry` | pure math: points, rects, paths, 4×4 matrices, strokes, color. No GPU, no unicode, no deps beyond glam |
| `valo-dl` | recording: `DisplayListBuilder`, ops, paints, shaders. GPU-free and `Send + Sync` |
| `valo-text` | typographer: shaping, bidi, wrapping, glyph raster, SDF, COLR. GPU-free |
| `valo-renderer` | the wgpu core: planner, encoder, pipelines, atlases, pools, caches |
| `valo` | the facade hosts use, plus `Hud` |
| `valo-svg` | SVG → display list translation |
| `valo-system-fonts` | native OS font discovery behind `FontSource`. Never a wasm dependency |
| `valo-capi` | C ABI for non-Rust embedders; the committed header is `crates/valo-capi/include/valo.h` |
| `valo-web` | wasm-bindgen bindings: the raw API, canvas attach, image upload. Ships to npm as `valo-web`; the `webgl` feature builds the WebGL2-fallback compat artifact |
| `valo-web-demo` | dev only: the browser playground chapters (`npm run dev:web`) |
| `valo-harness` | dev only: headless GPU, golden compare, example runner. Never a dependency of a shipping crate |

## Rules

Each rule exists for a reason; the reason is stated so you can tell when a rule genuinely does not apply, rather than following it blindly.

1. **Use wgpu directly — never wrap it in another GPU abstraction.** WebGPU is already a portable command encoder; a second portability layer would cost performance and clarity while buying nothing.

2. **Let specialist libraries do specialist jobs; valo owns the coordination between them.** Text shaping is harfrust, font parsing skrifa, glyph rasterising swash, line breaks and bidi the unicode crates, atlas packing etagere. valo owns recording, render planning, clipping, blur, blending, paragraph layout, and every cache. Why: those libraries each encode years of edge cases we should not re-learn — and the coordination between them is exactly the part nobody else provides.

3. **Compute drawing facts at record time, not at replay time.** Bounds, clip lifetimes, depth slots and layer bounds are stored on each recorded operation; replay just reads them. Why: recording sees the whole command stream with its clip stack live, so it can know these things cheaply — replay would have to re-derive them per frame, every frame, from less context. If replay is calculating something the recorder could have known, move the calculation.

4. **`cargo test` must stay headless — no browser, no display server.** Browser suites exist (Canvas2D conformance, site smoke, WebGL-fallback smoke) but are separate npm suites. Why: the Rust suite is the inner loop for engine work; if it ever needs a browser or a window, iteration speed and CI portability both die.

5. **The host supplies every external resource and render target.** Fonts arrive as registered bytes, images arrive uploaded, the frame target arrives from the host; valo never opens font files, touches the network, or owns a window. Why: every embedder — a game, an editor, a browser page, a C host — already has its own IO and window stack, and an engine that reaches around it cannot be embedded cleanly.

## Design facts

Not rules — facts about how the engine works that are cheap to know and expensive to rediscover. The detailed mechanics live as comments beside the code they describe.

- Coordinates start at the top-left and y grows downward, in logical pixels until a transform says otherwise (the Canvas2D/Skia convention).
- The public transform is a full 4×4, but matrix z never controls draw order. Depth is internal: the renderer assigns z to make clips depth-tested, and callers never see it. (The formula lives beside `slot_z` in `planner/replay.rs`.)
- Draws render into 4-sample scratch textures that hardware-resolve into single-sample persistent targets. The scratch is kept only when a later render section resumes the same target; the final section discards it, which lets tiled GPUs (phones, Apple Silicon) skip writing it to memory.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [seedeai/valo](https://github.com/seedeai/valo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
