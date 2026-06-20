---
trigger: always_on
description: Recreate a reference raster diagram/image in Microsoft Visio as editable native Visio shapes using pixel-coordinate mapping, sampled color palettes, Visio COM automation, layer ordering, text sizing, complex shape primitives, 2.5D/3D stacked diagram reconstruction, crop comparisons, and preview exports. Use when the user says visio_copy, visio-copy, asks to draw a PNG/JPG/PDF into a .vsdx, asks to redraw a diagram in Visio, asks to improve color fidelity, asks to reproduce complex or stacked sha
---


# Visio Copy

## Purpose

Use this skill to redraw a reference image into a real Visio diagram. The result must be editable native Visio shapes, not a pasted screenshot, not a source-image underlay, not an SVG/PDF import, not an auto-vectorized copy, and not any other tracing artifact.

This skill is optimized for technical diagrams with boxes, arrows, buses, labels, grids, clock-domain regions, paper-style color fills, shadows, gradients, and 2.5D/3D stacked blocks.

The workflow is Visio-first. Use analysis scripts only to inspect color, geometry, and crop differences; do not use them to create a rasterized final drawing. The reference image may be inspected outside the final Visio page for measurement and auditing, but it must not be inserted as a tracing underlay or remain as visible content.

## Artifact Safety Contract

- `redraw.vsdx` is reserved for a manual or manually reviewed Visio redraw that is acceptable as the current final artifact.
- Automated detection, batch reconstruction, or first-pass shape scaffolds must use names like `scaffold.vsdx`, `auto_first_pass.vsdx`, or `preview.png`; they must not overwrite `redraw.vsdx`.
- `redraw.vsdx` must be made of editable Visio shapes, connectors, text, and layers. Do not deliver a pasted source image, screenshot overlay, pixel-traced raster, or SVG/image conversion as the final redraw.
- Manual redraw scripts should open Visio visibly by default and keep the document open after saving so the drawing process is inspectable. Use `[bool]$Visible = $true` and `[bool]$KeepOpen = $true` in new manual `draw.ps1` scripts; pass `-Visible:$false -KeepOpen:$false` only for deliberate background/batch runs.
- Manual redraw workflows must update `preview.png` in the delivery directory after every accepted draw run. Prefer `scripts/export_visio_png_safe.ps1` after a hidden/closed run, or deliberately sync the latest preview from the active iteration directory. Do not rely on inline `Page.Export()` in visible keep-open scripts unless it has been tested for that figure; it can hang and leave the user thinking drawing failed.
- High-fidelity redraws are not the same as semantic architecture redraws. If the user asks for maximum restoration, treat pixel alignment as a deliverable: use 1:1 export scale, shared grid lines, per-component bboxes, and crop thresholds before claiming the figure is done. The source image may be inspected externally for measurement and audit, but must not be placed on the Visio page as a tracing underlay.

## Fidelity Modes

This skill has one mandatory delivery mode: `strict_visio_drawn`.

`strict_visio_drawn` means every visible element in the delivered `.vsdx` is created by Visio as an editable shape: rectangles, lines, connectors, polygons, tables, logic/electrical symbols, text, arrows, hatch lines, gradients built from Visio shapes, and grouped native shapes. The drawing process should be visible in Visio by default.

Forbidden in all normal use:

- placing the source PNG/JPG/PDF on the Visio page as a tracing underlay;
- importing SVG/PDF/vector traces;
- raster-to-vector conversion;
- pasted screenshots or image crops;
- auto-vectorized pixel rectangles;
- hiding poor redraw quality by leaving a source image underneath;
- switching to a trace workflow because the native Visio redraw is difficult.

The source image may only be used outside the final Visio page as a reference for measurement, color sampling, crop auditing, and visual comparison. If a task cannot be completed at the requested fidelity using native Visio shapes, say that explicitly and continue improving component descriptors and primitives rather than using tracing.

`semantic_editable` and `high_fidelity` are quality targets inside `strict_visio_drawn`, not alternate methods:

- `semantic_editable`: native Visio shapes preserve meaning and approximate layout.
- `high_fidelity`: native Visio shapes preserve source geometry as closely as possible. Every major component must be driven by source-image bboxes and crop audits, not by estimated module placement.

In `high_fidelity` mode:

- Initialize the page with `1 px = 1/96 in` unless there is a verified reason not to. This makes Visio's usual 96-DPI PNG export match the source raster size and avoids half-pixel resampling drift.
- Do not place the source image on the Visio page. Inspect it externally for measurements and crop audits, then draw native Visio shapes on a clean page.
- Draw repeated grids with shared internal lines. Do not draw each cell as a bordered rectangle when adjacent cells share edges; repeated borders create double-thick and uneven lines after export.
- For dense modules, first capture a descriptor: parent bbox, child bboxes, line weights, text lanes, ports, arrow endpoints, and draw order. Do not patch dense diagrams by page-global nudging.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zwj276765037-lab/Visio-copy](https://github.com/zwj276765037-lab/Visio-copy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
