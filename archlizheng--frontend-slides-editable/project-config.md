---
trigger: always_on
description: Use when the user wants a single-file HTML presentation that stays editable in the browser after generation, or needs object-level layout editing, slide reordering, local save/export, or PPT/PDF-to-web conversion with continued editing.
---


<objective>
Frontend Slides (Editable)

Create zero-dependency, animation-rich HTML presentations that run entirely in the browser **with a built-in editor**: move objects, multi-select with **Ctrl+click**, alignment snapping, simple text formatting with **font family + size controls**, **undo/redo**, a **Pages** sidebar (slide thumbnails, drag to reorder, delete), **Ctrl+S** persistence, and **export HTML**.

This skill is a **copy of the `frontend-slides` skill** extended with the editable deck runtime. For read-only decks without editor weight, use the original **`frontend-slides`** skill instead (same skills directory layout).
</objective>

<preset_fidelity>
Parity with parent `frontend-slides` (style flexibility)

**Adding edit mode must not replace preset authoring.** The parent skill treats each choice in [STYLE_PRESETS.md](STYLE_PRESETS.md) as a **spec**: per preset you implement its **Layout** prose, **Signature Elements**, typography, and colors — title slides and content slides **differ across presets** (e.g. Bold Signal’s card + big numerals vs. Notebook Tabs’ paper + edge tabs vs. Swiss Modern’s grid + red bar).

**Normative behavior in Phase 3:**

1. **Read STYLE_PRESETS for the chosen preset(s)** and reflect **layout + signatures** in HTML/CSS, not only `:root` colors and fonts.
2. **The editable reference** ([examples/editable-deck-reference.html](examples/editable-deck-reference.html)) supplies **JS/CSS patterns** (chrome, sidebar, objects, history) — not a **frozen slide layout** to paste on every deck. Do **not** reuse one generic “title + subtitle + corner rounded rectangle” geometry for every style.
3. **`examples/generated/presets/*.html`** are sample decks, not a layout shortcut. The 12 legacy samples are runtime smoke + visual previews; the 34 ported samples are real-template ports from `beautiful-html-templates` using the shared Swiss/reference runtime plus locked slot editing. For real deliveries, match the chosen preset/template grammar instead of reusing a generic README deck.
4. **Static chrome** (decorations that should not be draggable) may live outside `.slide-edit-layer` (e.g. background pseudo-elements, fixed nav chrome) per preset; **movable** copy and blocks stay as `[data-slide-object]` inside the layer.

If runtime constraints ever conflict with a signature element, **adapt the element** (e.g. implement the same visual with CSS, or split into multiple objects) — do **not** drop preset identity for the sake of a single template.
</preset_fidelity>

<template_ports>
Real-template ports (slot-editable)

For the 34 ported presets from `beautiful-html-templates`, prefer the upstream template's **mood / tone / density** and visual grammar over color-token matching alone.

Interaction baseline: ported decks must use the same Swiss/reference editor chrome and object editor as `examples/editable-deck-reference.html` / `swiss-modern.html`. Native template slots are locked-layout content slots; user-added objects are normal `[data-slide-object][data-oid]` objects.

**Important interaction contract:** `data-edit-slot` is editable content, not a draggable component. In edit mode the user can click slot text/images to change their content with RTE/Undo/Redo/Save/Export, but the upstream template's layout grid, card geometry, decorative layers, and spacing stay locked. Only user-added objects in `.slide-edit-layer` become selectable, draggable, resizable components. Do not promise that every native template child will get object handles unless a separate componentization mode is explicitly requested and implemented.

Template edit modes:

- `data-template-edit-mode="slots"` is the default. It is template-safe: authored content is editable through slots, while layout and decoration remain locked.
- `data-template-edit-mode="components"` is an optional generated mode for semantic blocks that can be represented as `[data-slide-object]`.
- In delivered ported decks, **Unlock layout** componentizes the current slide on demand by creating movable copies of editable slots in `.slide-edit-layer`. The original native template DOM stays in place for fidelity, and the operation is undoable.
- Never componentize every DOM node. Exclude backgrounds, grids, axes, ticks, decorative marks, texture/glitch layers, SVG paths, animation wrappers, and pure layout containers.

When using or extending a ported preset:

1. Treat the matching `beautiful-html-templates/templates/{source_slug}/template.html` as the visual system: preserve fonts, CSS variables, slide-level classes, layout grid, decorative DOM, and component grammar.
2. Edit authored content through slots, not by decomposing the template into draggable boxes. Use:
   - `data-edit-slot`
   - `data-slot-type="text|image|metric|table-cell"`
   - `data-slot-label`
   - `data-slot-locked-layout="true"`
3. Keep decorative elements locked unless the user should naturally change that exact item. Grids, paper texture, hairlines, scan lines, pixel/glitch layers, ornamental marks, and layout containers should not become draggable objects.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [archlizheng/frontend-slides-editable](https://github.com/archlizheng/frontend-slides-editable) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
