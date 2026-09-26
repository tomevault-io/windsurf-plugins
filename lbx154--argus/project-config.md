---
trigger: always_on
description: The visual tokens below are aligned to the authoritative Argus
---

# Argus Figure Studio v2 — shared conventions (read before editing anything in studio/)

The visual tokens below are aligned to the authoritative Argus
`figure_tool.py` template `argus-image2-paper-prompt-v1`.

Goal: one integrated, automated pipeline for CCF-A paper figures that combines the
Argus drawing skills that today live in separate places:

| Argus skill / asset | What we take from it |
|---|---|
| Research Visualization Router | route by semantics: data chart → Matplotlib/SciencePlots; conceptual/method/architecture → native editable PPTX via PPT Master |
| Paper Framework Figure Studio | one-sentence takeaway → exact modules/labels/connections → editable source + vector export |
| **"Figma-style" design system** (old `figure_tool.py` PAPER_FIGURE_PROMPT_TEMPLATE, removed from current Argus) | rounded cards rx 10–16, dark-gray strokes `#1f2933`, soft pastel semantic fills, compact density, numbered step badges, phase containers, chips, section tabs, no shadows/gradients/glassmorphism, 20 layout variants |
| PPT Master (installed, `~/.argus-skill/tools/ppt-master/skills/ppt-master`) | SVG authoring contract, `svg_quality_checker.py`, `svg_to_pptx.py` (native DrawingML), `pptx_to_svg.py` round-trip, `templates/icons/tabler-outline/*.svg`, `templates/charts/*.svg` as layout references, visual style `soft-rounded` |
| FigureSpec | deterministic JSON → SVG, machine-checkable geometry |
| Paper Chart Styling (`figure_spec_scripts/paper_chart_style.py`) | `set_pub_style / figure_size / highlight_ours` for the data figure |
| B-group `optimized/paper_figure_renderer.py` + `figure_quality_gate.py` | orthogonal routing, boundary ports, label pads, deterministic gate |

## Interpreter
Always `/data/v-boxiuli/argus_test_env/bin/python` (has python-pptx, Pillow, matplotlib, scienceplots, cairosvg, lxml).
PPT Master scripts: `PM=/home/v-boxiuli/.argus-skill/tools/ppt-master/skills/ppt-master`.

## Canvas and publication scale (hard rule)
- Default canvas `viewBox="0 0 1280 720"` (PPT Master `ppt169`) — the only canvas `svg_to_pptx.py -f ppt169` accepts. A flat figure root declares `data-pptx-page-role="content"`; its first visible child is the full-canvas warm-white background `<rect id="background" data-pptx-role="background" fill="#FBFAF7"/>`.
- Every contract declares `final_width_mm` (178 = double-column `figure*`, 84 = single column). Physical type is `font_px × final_width_mm / 1280 / 0.3528` pt. **Minimum 8 pt for every visible role**; at 178 mm the integer floor is 21 px (8.278 pt). The strict hierarchy is 21 px chips/labels/badges/legends/footnotes/sublabels, 24 px card labels, 27 px section/group labels, and 30 px optional page titles. Never shrink any emitted text below the floor to make geometry fit.
- Background token `#fbfaf7` always; edge-label masks, group-label masks, and icon slots use the same warm white so no white patch appears on the canvas. True white is reserved for badge-number text.

## Figma-style tokens (`studio/figma_tokens.py` owns them; nobody else hard-codes colours)
- Stroke `#1f2933` (cards 2 px, connectors 2 px), text `#111827`, secondary text `#4b5563`, group border `#9ca3af` dashed.
- Semantic pastel fills: `input/data #ffe2d1`, `process/compute #fff2bd`, `memory/storage #dcecff`, `agent/model #e2f7df`, `output/eval #eadfff`, `benchmark/metric #fff1c9`, `neutral #f3f4f6`. Highlight ("ours"/optimal path) `#d55e00` 3 px. Colour-blind-safe check: shapes/line styles must also encode meaning, not colour alone.
- Cards `rx=12`; card padding, pill padding/height, badge diameter, and badge gap are derived from their active type role in `figma_tokens.py`, with horizontal card padding never below 12 px. Pills/chips use full radius; badges use white bold minimum-scale numbers.
- Fonts: `font-family="Helvetica, Arial, 'Liberation Sans', sans-serif"`; weights 400/600/700 only.

## Layer corridors and stores
- Layer gaps are derived from corridor contents, never treated as a cosmetic fixed gap: source-to-bus clearance is at least 16 px and every marker-ended final segment is at least 28 px. Group padding, the group-label chip plus its 10 px routing obstacle, and centred edge-label pills are added to that core reservation.
- When the canvas is exhausted, layered layout compacts in this order: horizontal card padding (never below 12 px), then compact title wrapping (LR layouts only), then discretionary natural gap, then node-title type; type stops at the 21 px publication floor and a still-unfittable layout fails. Text widths are estimated from Helvetica AFM advances with a separate bold table (`figma_tokens.text_width_px`); the estimate must stay within about 3% of the rendered TeX Gyre Heros width so cards and pills are sized from real glyph metrics. Arrow and group-label clearances do not shrink. A label never changes its edge route: if a measured pill cannot fit inline, place it perpendicular to a straight segment with a 6 px, 1.5 px-wide `GROUP_BORDER` leader tick.
- A `store` is a standard-height rounded card with a second same-size card offset 4 px down/right behind it. The node's `data-pptx-bounds` encloses both cards; stores do not use interior divider lines.

## PPT Master SVG contract (must pass `$PM/scripts/svg_quality_checker.py <svg> --format ppt169` with 0 errors)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lbx154/Argus](https://github.com/lbx154/Argus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
