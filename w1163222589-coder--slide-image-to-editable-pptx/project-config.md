---
trigger: always_on
description: >
---


# Slide Image → Editable PPTX

You are converting slide screenshots into editable PowerPoint files.
The goal is **visual fidelity** — the output PPT, when rendered, should look
nearly identical to the source image — while making text and geometric shapes
editable.

## Core Principle: Three-Layer Decomposition

Every pixel in the source image belongs to exactly one of three layers:

| Layer | What it contains | How to implement | Editable? |
|-------|-----------------|------------------|-----------|
| **A — Visual Asset** | Complex illustrations, decorative backgrounds, photos, scientific figures, icons with intricate detail, maps, radar scenes, waveforms, heatmaps, campus sketches, device drawings, textured decorations | Clean PNG generated with `$imagegen` (no text!) or original figure preserved as image | Replaceable (move/resize/delete), not internally editable |
| **B — Structure** | Solid color rectangles, rounded rectangles, circles, lines, arrows, dividers, panel frames, card backgrounds, badges, gradient bars, simple geometric decorations | PPT-native shapes via `$slides` / `presentation-skill` (PptxGenJS) | Fully editable |
| **C — Content** | All readable text: titles, subtitles, body text, labels, captions, page numbers, footer text, formula text, table text, card headings, legend labels | PPT-native text boxes via `$slides` / `presentation-skill` (PptxGenJS) | Fully editable |

**The key insight**: Never bake text into a generated image. Never use crude PPT shapes to approximate a complex visual. Never use a full-slide screenshot as background.

## Workflow

### Phase 1: Pixel-Level Analysis

For EACH source image, produce a structured analysis. Do not skip any slide. Do not infer from one slide what another contains — inspect each one.

#### Step 1.1: Observe and catalog

Look at the image carefully. Identify every distinct visual element. For each element, determine:

```
{
  "element_id": "s01_e01",
  "description": "distributed radar network illustration with 6 antennas around a central target on a coordinate grid",
  "bbox_percent": {"x": 45, "y": 10, "w": 55, "h": 80},
  "layer": "A",
  "implementation": "image2-generate",
  "z_order": 1,
  "notes": "Must NOT include any text labels. The title, axis labels, and page number are separate text elements."
}
```

The `bbox_percent` uses percentages of slide width/height (0-100) to specify position. This avoids pixel-count errors across different image resolutions.

#### Step 1.2: Classify elements strictly

Apply these rules in order:

1. **Is it readable text?** → Layer C (text box). This includes ALL text: titles, labels inside diagrams, axis labels, page numbers, footer text, card headings, bullet points, formula text. Even if text overlaps a complex visual, the text itself goes to Layer C.

2. **Is it a simple geometric shape?** (solid rectangle, rounded rectangle, circle, line, arrow, triangle, chevron, trapezoid — with solid fill or simple border) → Layer B (PPT shape). This includes panel backgrounds, card frames, title bars, dividers, badges, progress bars, simple decorative strips.

3. **Is it visually complex?** (illustration, photo, texture, gradient background with imagery, scientific figure, diagram with intricate line work, icon with detail beyond basic geometry) → Layer A (generated PNG or preserved figure).

#### Step 1.3: Identify shared vs. unique elements

- **Shared across slides**: header bar style, footer style, logo, page number format, background color
- **Unique per slide**: main content visuals, specific diagrams, data figures

Create a `slide_master_elements` list for shared elements, and per-slide element lists for unique content.

#### Step 1.4: Completeness self-check (required)

After finishing the element list for ALL slides, go back and re-examine each source image ONE MORE TIME with fresh eyes. This second pass focuses specifically on **small or easy-to-miss visual elements** that the first pass may have overlooked.

For each slide, ask yourself these questions:

1. **Small icons**: Are there any small icons (university seals, bullet-point icons, award badges, folder icons, chip icons, person icons, book icons, etc.) that I classified as Layer B (PPT shape) but are actually too detailed for simple shapes? If the icon has more than 3-4 visual features (gradients, shadows, internal detail, curves), reclassify it as Layer A.

2. **Decorative details**: Are there small decorative elements I skipped entirely? Look in corners, edges, between cards, along dividers, and in footer/header areas. Things like: small wave patterns, dot grids, circuit-trace textures, line-art campus buildings, faint background motifs.

3. **In-card visuals**: For each card/panel on the slide, does it contain a small illustration or diagram inside it? These are frequently missed because the analyst focuses on the card frame (Layer B) and card text (Layer C) but forgets the small visual inside the card.

4. **Chart/figure decorations**: Around charts or data figures, are there small visual elements like target icons, antenna icons, signal-path illustrations, or comparison diagrams that I might have grouped with the chart but are actually separate Layer A elements?


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [w1163222589-coder/slide-image-to-editable-pptx](https://github.com/w1163222589-coder/slide-image-to-editable-pptx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
