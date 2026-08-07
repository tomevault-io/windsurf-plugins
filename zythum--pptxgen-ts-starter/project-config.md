---
trigger: always_on
description: > Build `.pptx` presentations with JSX + TypeScript + pptxgenjs. Slides are JSX components compiled into PowerPoint files.
---

# pptxgen-ts-starter — Agent Guide

> Build `.pptx` presentations with JSX + TypeScript + pptxgenjs. Slides are JSX components compiled into PowerPoint files.

## Tech Stack

| Layer            | Technology                                                                                    |
| ---------------- | --------------------------------------------------------------------------------------------- |
| JSX Components   | `@zythum02/pptxgenjsx` — `<Slide>`, `<Text>`, `<Rect>`, `<Chart>`, `<Table>`, `<Image>`, etc. |
| Rendering Engine | `pptxgenjs` ^4.0.1                                                                            |
| Dev Server       | Vanilla Node HTTP via `tsx` — **no file watching, refresh browser manually**                  |
| PPTX Viewer      | `@silurus/ooxml` (browser)                                                                    |
| TS Config        | `jsx: "react-jsx"`, `jsxImportSource: "@zythum02/pptxgenjsx"`                                 |

## Project Structure

```
src/
├── ppt.tsx                  # Entry — default export, composes all slides in <Deck>
├── slides/                  # One file per slide, numbered: 01-title.tsx, 02-*.tsx …
├── components/              # Shared UI: SlideBackground, SectionHeader, Card
├── token/
│   ├── colors.ts            # Color tokens — single source of truth for slide colors
│   └── typography.ts        # Font & size tokens — single source of truth for slide typography
└── media/images/            # Image assets — paths relative to src/ppt.tsx
scripts/
├── color-tool.ts            # Derive palette variants + check WCAG contrast
├── dev-server.ts            # Dev server — do not modify
├── estimate-text.ts         # Measure rendered text height (prevent overflow)
├── generate-pptx.ts         # .pptx builder — do not modify
└── image-tool.ts            # Query image metadata, crop, resize
web/index.html               # Browser PPTX viewer
.agents/skills/pptxgenjsx/   # Component API reference (loaded on demand)
.agents/skills/design/       # Design guidance: workflow + templates & themes
.deck/                       # Per-deck workspace: brief.md / research.md / outline.md / spec.md
output/                      # Generated .pptx files
```

## Conventions

### Slides

- **Default async exports (anonymous)** — `export default async function ()`. Components are regular functions (`export function Card()`). **Never annotate return types.**
- **Numbered files** — `01-title.tsx`, `02-overview.tsx`, ordered by presentation flow.
- **Full-size background first** — every slide's first child is `<SlideBackground />`.
- **Composed in `src/ppt.tsx`** — import all slides, arrange inside `<Deck>`.

### Components

- Extract into `components/` when a pattern appears in **3+ slides**.
- Expose only what varies — `<SlideBackground color="dark" />` takes zero position props; `<Card>` takes all four (`x, y, w, h`).
- Keep positioning visible in the slide file, not hidden behind a component abstraction.

### Positioning & Canvas

- **Absolute inches** — every element needs `x`, `y`, `w`, `h`. No flexbox, no grid.
- Canvas size — `<Deck layout={...}>` accepts a built-in enum (e.g. `"LAYOUT_WIDE"`) or a custom `{ width, height }` object. For enums, look up dimensions in `.agents/skills/pptxgenjsx/references/deck.md` (built-in layout table).

### Colors

- Hex **without `#`** — `"7C3AED"` not `"#7C3AED"`.
- Transparency: `fill: { color: "18181B", transparency: 50 }` (0–100).
- **Single source of truth** — all colors used in slides must come from
  `src/token/colors.ts` (`colors.background`, `colors.primary`, …). Never
  hardcode bare hex in slide files. Fill the token file from a chosen palette
  (see `.agents/skills/design/templates-themes/palettes.md`) when starting a
  deck; derive new variants with `scripts/color-tool.ts`, then register them
  in `colors.ts` under a semantic name.

### Typography

- **Single source of truth** — `fontFace` / `fontSize` come from
  `src/token/typography.ts` (`typography.font.*`, `typography.size.*`) — no
  magic numbers in slides.
- `bold: true/false` is written directly — pptxgenjs has no numeric
  `fontWeight`.
- New sizes: update `.deck/spec.md` §4 first, then propagate to `typography.ts`.
- Per-slide exceptions (measured with `estimate-text.ts`) may use literal
  values, with a comment explaining why.

### Shapes

| Component     | Use when                                                            |
| ------------- | ------------------------------------------------------------------- |
| `<Rect>`      | Plain rectangle — backgrounds, accent bars, dividers                |
| `<RoundRect>` | **Visible** rounded corners — cards, badges. Never `rectRadius={0}` |
| `<Ellipse>`   | Circles and ellipses                                                |

### Code Style

- **No return type annotations** — let TypeScript infer from JSX.
- `src/ppt.tsx` must be a **default export** (`export default function ()`).
- **No HTML elements** — `<div>`, `<span>`, `<p>` are not in `JSX.IntrinsicElements`. Only imported components are valid.
- **Don't modify** `scripts/*` `web/*` — infrastructure, not content.

## Workflow: Add or Edit a Slide


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zythum/pptxgen-ts-starter](https://github.com/zythum/pptxgen-ts-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
