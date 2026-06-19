---
trigger: always_on
description: Generate professional, publication-ready scientific figures as editable SVG files for academic journals (Nature, Science, Cell, ACS, IEEE, Elsevier). Use this skill whenever the user wants to create a schematic figure, graphical abstract, TOC graphic, experimental workflow diagram, conceptual illustration, multi-panel figure, or any visual for a research paper or presentation. Also trigger when users mention "figure for paper", "journal figure", "scientific illustration", "panel figure", "graphi
---


# Academic Figure Generator

Generate publication-ready, editable SVG figures for top-tier academic journals. Outputs are directly editable in **Inkscape** and **Ipe** (via SVG import).

## Core Workflow

```
User provides research story/idea
        ↓
Analyze → identify panels, flow, hierarchy
        ↓
Determine figure type (multi-panel figure OR graphical abstract/TOC)
        ↓
Design 2-3 variants (different layouts AND visual styles)
        ↓
Generate SVG files with:
  - Clean vector graphics (shapes, arrows, labels)
  - Labeled placeholder regions for data plots
  - Journal-compliant typography and dimensions
        ↓
Present all variants for user to choose/refine
```

---

## Step 0: Read References

**BEFORE generating any SVG**, read the reference files:

1. **Read** `references/journal-styles.md` — journal dimension/formatting rules
2. **Read** `references/svg-guide.md` — SVG generation patterns, component library

These contain critical information about exact dimensions, fonts, color palettes, arrow spacing rules, and reusable SVG component code. Do NOT skip this step.

---

## Step 1: Analyze the User's Story

When the user describes their figure, extract:

1. **Narrative arc**: What story does the figure tell? (e.g., "We synthesized X, characterized it with Y, and it performs Z")
2. **Panel inventory**: How many distinct panels? What does each show?
   - **Schematic panels**: Diagrams, workflows, device structures, molecular illustrations
   - **Data panels**: Plots, spectra, images (these become labeled placeholders)
   - **Annotation panels**: Scale bars, legends, color bars
3. **Reading order**: Left-to-right? Top-to-bottom? Circular workflow?
4. **Target journal** (if specified): Determines dimensions, fonts, column widths
5. **Figure type**: Determine from context:
   - **Multi-panel research figure** — the default. Panels labeled a, b, c... with schematics and data placeholders.
   - **Graphical abstract** — single unified visual summarizing the paper, no panel labels, more iconic/illustrated style.
   - **TOC graphic** — very small (ACS: 82.55 × 44.45 mm), high-impact thumbnail, minimal text, bold visual.

If the user hasn't specified a journal, default to Nature-style formatting (widely compatible).

### Asking Clarifying Questions

If the user's description is ambiguous, ask ONE focused question. Do not overwhelm with multiple questions. Prefer making reasonable assumptions and noting them.

---

## Custom Color Schemes

Users can supply their own color scheme, which overrides the default style palette. Accept colors in any format (hex, RGB, color names) and normalize to hex for SVG output.

### How Users Specify Colors

Users may say things like:
- "Use blue (#2166AC) and red (#B2182B) as the main colors"
- "Match my lab's brand colors: navy, gold, and white"
- "Use the ACS style colors"
- "I want a monochrome figure in shades of gray"
- "Use the same palette as my matplotlib viridis plots"

### Color Scheme Structure

When the user provides custom colors, map them into these roles:

```python
custom_scheme = {
    'accent1': '#...',    # Primary color (boxes, main arrows, key elements)
    'accent2': '#...',    # Secondary color (secondary elements, highlights)
    'accent3': '#...',    # Tertiary color (supporting elements)
    'accent4': '#...',    # Quaternary (optional, for 4+ category figures)
    'bg': '#FFFFFF',      # Figure background (default white unless specified)
    'text_color': '#...',  # Auto-derived: black for light bgs, white for dark
    'arrow_color': '#...', # Auto-derived: darkened version of accent1
}
```

### Auto-Derivation Rules

If the user provides only 1-2 colors, derive the rest:

- **1 color given**: Use it as accent1. Generate accent2 as a 30° hue-shifted variant, accent3 as a 60° variant. Keep backgrounds white.
- **2 colors given**: Use as accent1 and accent2. Derive accent3 by interpolating or shifting hue.
- **3+ colors given**: Map directly to accent1, accent2, accent3, accent4.
- **Background**: Default white unless the user specifically requests a colored or dark background.
- **Text color**: Use #222222 (near-black) for light backgrounds, #F0F0F0 for dark backgrounds.
- **Arrow color**: Use a darkened (70% luminance) version of accent1, or #444444 if accent1 is too light.
- **Placeholder fill**: Use a very light tint (10% opacity) of accent1.

### Applying Custom Colors to Variants

When custom colors are provided, ALL variants use the same color scheme (since the user wants these specific colors). The variants still differ in layout and structural style (line weights, corner radii, spacing), but use the same palette.

However, if the user says "give me different color options too", then vary the color application across variants:
- Variant A: Colors used minimally (outlines only)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sai-tv/academic-figures](https://github.com/sai-tv/academic-figures) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
