---
trigger: always_on
description: Quick-start orientation for AI assistants and new contributors.
---

# GUI Framework — Developer Reference

Quick-start orientation for AI assistants and new contributors.
Framework: `gui/` module, immediate-mode declarative UI in V.

## Framework Overview

Immediate-mode pattern — every frame rebuilds the UI from a pure function.

Key types:

- **`Window`** (`window.v`) — owns the render loop, layout tree, animations,
  IME, A11y, and command queue. One per app.
- **`View`** (`view_*.v`) — interface; user-facing config structs that implement
  `generate_layout(mut Window) Layout`.
- **`Layout`** (`layout*.v`) — resolved node tree: shape + children. Output of
  `compose_layout`. Discarded each frame (full rebuild) or amended
  (render-only path).
- **`Shape`** (`shape*.v`) — drawing descriptor: geometry, color, optional
  sub-structs `tc &TextConfig`, `fx &ShapeEffects`, `&EventHandlers`.
- **`Renderer`** / `renderers_draw` — flat list of draw commands emitted by
  `render_layout`. Consumed once per frame by `renderers_draw`.

Example program convention: the first container must be `gui.fixed_fixed`
with width and height matching the window dimensions.

Entry point pattern:

```v ignore
gui.window(gui.WindowCfg{
    on_init: fn (mut w gui.Window) { w.update_view(my_view_fn) }
})
```

## Render Pipeline

```
frame_fn
  flush_commands          // thread-safe state mutations
  init_ime / init_a11y    // lazy first-frame init
  if refresh_layout       // full rebuild (view fn + layout + renderers)
    update()
      view_generator()    // calls user view fn
      compose_layout()    // View tree → Layout tree
        generate_layout   // View → Layout nodes
        layout_arrange    // size/position passes (amend_layout fires here)
      rebuild_renderers() // Layout tree → flat []Renderer
  else if refresh_render_only  // renderer rebuild only (layout reused)
    update_render_only()
      rebuild_renderers()
  process_svg_filters     // offscreen passes before swapchain
  renderers_draw()        // draw flat renderer list
```

Trigger full rebuild:   `w.update_window()` sets `refresh_layout = true`
Trigger render-only:    `w.rerender_window()` sets `refresh_render_only = true`
`refresh_layout` takes priority over `refresh_render_only`.

When to use each path:

| Condition | Path |
|-----------|------|
| View state changed (app data, focus, etc.) | `refresh_layout` |
| Animations with `render_only: true` | `refresh_render_only` |
| Cursor blink, progress bars | `refresh_render_only` |
| Tween / Spring / Hero transitions | `refresh_layout` |

**Important**: layout/hero transitions lerp from a snapshot to the current
position. Repeated `rerender` corrupts the interpolated values — use full
rebuild.

## Layout Pipeline

```
compose_layout(mut view)
  generate_layout(mut view, mut window)   // View → Layout (recursive)
  layout_arrange(mut layout, mut window)  // 3-pass: size, amend, position
    layout_sizes()         // distribute grow/shrink space (layout_sizing.v)
    amend_layout callbacks // user hooks run HERE (not in render_layout)
    layout_positions()     // x/y assignment, scroll offset clamping
  wrap root in transparent Shape
```

`amend_layout` runs during `layout_arrange`, **not** during `render_layout`.
Use it to mutate layout geometry or inject child layouts.

`render_cursor` runs during `render_layout`, reads `input_cursor_on` live
(never captured in a closure).

## GC / Boehm False-Retention Rules

V uses the Boehm conservative GC. Key hazards:

### 1. `array.clear()` retains stale pointers

`clear()` sets `len=0` but does **not** zero the backing memory. The GC scans
the entire allocated block, so stale pointers in cleared arrays cause false
retention (objects never collected).

**Rule**: use `array_clear(mut arr)` (defined in `gc.v`) for any array containing
pointers or pointer-containing types. It calls `vmemset` before zeroing `len`.

```v ignore
array_clear(mut window.renderers) // NOT window.renderers.clear()
```

### 2. Closure capture — full struct pointer

`fn [cfg]` closures capture the entire `@[heap]` cfg struct pointer.
Conservative GC scans ALL pointer-sized words in the struct → false retention
proportional to struct size.

**Rule**: extract only the fields needed into locals, then capture those:

```v ignore
// Bad — captures entire InputCfg (many pointer-sized fields):
on_char: fn [cfg] (l &Layout, mut e Event, mut w Window) { ... }

// Good — extract minimal fields:
id_focus := cfg.id_focus
color_hover := cfg.color_hover
on_char: fn [id_focus, color_hover] (l &Layout, mut e Event, mut w Window) { ... }
```

Bound methods stored as callbacks (`cv.method_name`) are closures too —
they capture `cv` pointer. Convert to standalone functions.

Applied to: `view_input.v` (root cause), `view_select.v`, `view_menubar.v`,
`view_color_picker.v`, `view_container.v` (tooltip), `view_input_date.v`,
`view_table.v`.

See `make_input_on_char`, `make_select_on_keydown`, `make_menubar_amend_layout`
for the extraction pattern.

### 3. Layout scrubbing between frames

Old `Layout` nodes must be freed between frames. `layout_clear` zeros and
frees each node's `Shape`. Guard: never free `empty_layout.shape` (module
constant).

## Shape Struct Layout

`Shape` uses optional sub-structs to reduce per-shape memory footprint:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vlang/gui](https://github.com/vlang/gui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
