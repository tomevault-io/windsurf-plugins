---
trigger: always_on
description: Shared visualization rules: D2 diagrams, hvplot/bokeh, source images, priority order
---


# Visualization Standards

Shared visualization rules for all textbook chapter workflows (research, write, edit, update).

**Goal:** Your chapter should contain the absolutely perfect picture to explain each concept.

---

## Visual Priority Order (CRITICAL)

**Follow this order when choosing how to illustrate a concept:**

1. **Source images from downloaded papers** — Check the TEXTBOOK-PLAN.md Source Image Catalog FIRST. These are canonical, authoritative figures that readers expect to see. Copy them to `{Chapter}/images/` and embed.
2. **D2 diagrams** — for concept maps, flowcharts, and structural diagrams. Always use ELK engine.
3. **Python/hvplot (bokeh backend)** — for data visualizations, distributions, function plots, and ANY visual that must be numerically accurate.
4. **Web downloads** — for images not in sources/ (search and download during writing).
5. **generate_image** — ONLY for decorative/conceptual illustrations where numerical accuracy is irrelevant (e.g., a stylized icon, a non-data artistic illustration). See the warning below.

> **NEVER use `generate_image` for plots, charts, graphs, reliability diagrams, bar charts, heatmaps, confusion matrices, or ANY visual that needs to display accurate data.**
>
> LLM image generation tools (e.g. Gemini Imagen, DALL-E) produce visually plausible but **factually incorrect** data in plots. The numbers, axis labels, bar heights, curve shapes, and data points will look reasonable but will be WRONG. This is unacceptable in a textbook.
>
> **For any visual that contains numerical data, use one of these instead:**
> - **Source images from papers** — always preferred for canonical results (copy from `sources/`)
> - **Python code** — generate the plot programmatically with `hvplot` (bokeh backend) or `matplotlib` directly, using real data or carefully constructed synthetic data
> - **Web download** — find the original published figure online and download it
>
> The ONLY acceptable use of `generate_image` is for purely conceptual/artistic illustrations where no data accuracy is needed (e.g., a stylized banner image, an abstract concept illustration).

### Choosing the Right Visualization Approach

| Type of Visual | Approach |
|---|---|
| **Canonical figures from papers** (architecture diagrams, attention maps, scaling plots) | **Copy from `sources/` — see Source Image Catalog** |
| **Data plots** (distributions, functions, comparisons) | **Generate with code (hvplot with bokeh backend) — NEVER use `generate_image`** |
| **Reproducing a paper's plot** (when source image unavailable or low-res) | **Write Python code to recreate it from the paper's reported numbers** |
| **Simple concept maps** (flowcharts, relationships) | Generate with D2 |
| **Mathematical diagrams** (geometric, annotated) | Generate with TikZ |
| **Complex images NOT in sources** (rare) | **Download from web** |
| **Decorative/conceptual art** (no data accuracy needed) | `generate_image` (ONLY case where this is acceptable) |

---

## Source Images (From Downloaded Papers — PREFERRED)

1. Check the TEXTBOOK-PLAN.md **Source Image Catalog** for images assigned to this section
2. **CRITICAL: Always convert from PDF, never just copy the PNG.** arXiv source PNGs are frequently low-resolution thumbnails (e.g., 586x288px) or blank white placeholders. Even when a PNG exists and looks non-empty, it is almost always a low-DPI version of the PDF. The PDF is the authoritative source.
   ```bash
   mkdir -p "{Chapter}/images"
   pdf_source="AI-Learning-Gems/sources/arxiv-XXXX/figures/figure.pdf"
   png_source="AI-Learning-Gems/sources/arxiv-XXXX/figures/figure.png"
   dest="{Chapter}/images/descriptive-name.png"
   if [ -f "$pdf_source" ]; then
     magick -density 400 "$pdf_source" -flatten -trim +repage "$dest"
   elif [ -f "$png_source" ]; then
     cp "$png_source" "$dest"
     echo "WARNING: No PDF found, copied PNG directly. Verify resolution."
   fi
   ```
   **Why?** LaTeX `\includegraphics{figures/teaser}` picks the PDF (vector, full resolution). PNGs in arXiv sources are low-res fallbacks. Copying the PNG gives a thumbnail (e.g., 586x288); converting the PDF at 400 DPI gives a crisp figure (e.g., 2412x1161). macOS `sips` CANNOT rasterize vector PDFs. Always use ImageMagick (`brew install imagemagick ghostscript`).
3. Name descriptively: `vit-architecture.png`, `dino-attention-maps.png`, `scaling-vs-data.png`
4. Embed with caption and attribution:
   ```markdown
   ![Caption describing the figure. Source: Author et al. (Year), Figure N.]([Topic Name]/images/descriptive-name.png){#fig-label}
   ```
5. **CRITICAL:** Always attribute the source in the caption. Use the format: `Source: Author et al. (Year), Figure N.`
6. **CRITICAL:** Do NOT add explicit `width` tags to images (e.g. `width="60%"`, `width="95%"`). Omit the width entirely and let Quarto's global defaults handle the image sizing.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AI-Learning-Gems/AI-Learning-Gems.github.io](https://github.com/AI-Learning-Gems/AI-Learning-Gems.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
