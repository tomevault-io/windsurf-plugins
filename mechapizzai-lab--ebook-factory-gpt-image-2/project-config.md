---
trigger: always_on
description: >
---


# Ebook Factory — AI Visual Book Generator

Generate complete ebooks where **every single page is a full-page image** generated
by AI. All text content (titles, body text, ingredients, instructions, table of
contents, page numbers) is rendered directly inside the images by GPT Image 2.
The final book is an assembly of full-page images formatted via Calibre for
Amazon KDP. Output language is **French** by default.

## Core principle — FULL-IMAGE PAGES

**CRITICAL**: This skill produces books where every page — including the cover,
table of contents, section dividers, content pages, and back cover — is a single
full-page image. There is NO separate HTML text layer. All text that appears in
the book is part of the image itself, rendered by GPT Image 2.

This means:
- Every image prompt MUST include the exact text to render on that page
- Text must be in the correct language (French by default)
- Typography, layout, and text placement are controlled via the image prompt
- The EPUB/PDF is essentially a fixed-layout image album
- GPT Image 2's strong text rendering capability is essential for this workflow

## Overview

This skill orchestrates a multi-phase pipeline:
1. **Analyze** the user's request and select a book template
2. **Prototype** a sample page, cover, and table of contents for validation
3. **Iterate** based on user feedback until approved
4. **Produce** all pages as full-page images with consistent style
5. **Export** via Calibre to fixed-layout KDP-ready formats

## Supported book types

| Type | Template | Typical pages | Key features |
|------|----------|---------------|--------------|
| Cookbook / Recipe book | `templates/cookbook.md` | 30-80 | Recipe structure, ingredient lists, step photos |
| Manga / Comic | `templates/manga.md` | 20-60 | Panel layouts, speech bubbles, character consistency |
| Children's picture book | `templates/picturebook.md` | 16-32 | Full-page illustrations, minimal text, age-appropriate |
| Visual guide / How-to | `templates/visualguide.md` | 20-50 | Step-by-step photos, diagrams, annotations |
| Art book / Portfolio | `templates/artbook.md` | 24-48 | Full-bleed images, minimal text, gallery layout |

## Dependencies

### Required
- Python 3.10+
- Calibre (`ebook-convert` CLI) — for format conversion and KDP export
- ebooklib — for fixed-layout EPUB assembly (images only)
- Pillow — for image validation, resizing, and DPI verification

### NOT needed (no HTML text layer)
- ~~Jinja2~~ — no text templating, all text is in images
- ~~BeautifulSoup4~~ — no HTML content processing

### Image generation
- **Codex**: uses built-in `image_gen` tool (no API key needed)
- **Claude Code**: requires `OPENAI_API_KEY` for GPT Image 2 via `scripts/generate_image.py`

### Setup
```bash
# Install Python dependencies
pip install ebooklib Pillow

# Install Calibre (macOS)
brew install calibre

# Install Calibre (Ubuntu/Debian)
sudo apt-get install calibre

# Install Calibre (Windows)
# Download from https://calibre-ebook.com/download
```

## Phase 1 — Analyze

When the user requests a book, extract these parameters:

```
Book type:     <cookbook | manga | picturebook | visualguide | artbook>
Theme:         <e.g., "Alsatian recipes", "space adventure", "garden flowers">
Visual style:  <e.g., "cartoon", "watercolor", "anime", "photorealistic", "minimalist">
Language:      <default: French — all text rendered in images must be in this language>
Page count:    <target, or "auto" to let template decide>
Format:        <KDP print | KDP ebook | both>  (default: both)
Dimensions:    <auto based on format, or user-specified>
Page size:     <auto from KDP format — all generated images must match this exact size>
```

### Language rule

ALL text rendered inside images MUST be in French by default (or the language
specified by the user). This includes:
- Cover title and subtitle
- Table of contents entries
- Section/chapter titles
- Recipe names, ingredient lists, instructions
- Story text, dialogue, captions
- Page numbers
- "Sommaire", "Table des matières", "Préparation", "Ingrédients", etc.

When writing image prompts, always specify:
`"All text in the image must be in French: [exact text to render]"`

### Page dimensions rule

Every generated image must have the EXACT same pixel dimensions, matching
the target book format. This ensures consistent assembly:

| Format | Image dimensions | Aspect ratio |
|--------|-----------------|--------------|
| KDP Ebook (Kindle) | 2560×1600px | 16:10 landscape |
| KDP Ebook (portrait) | 1600×2560px | 10:16 portrait |
| Print 6×9" (300 DPI) | 1800×2700px | 2:3 portrait |
| Print 8.5×11" (300 DPI) | 2550×3300px | ~3:4 portrait |

Store the chosen dimensions in `project_config.json` and use them for
EVERY image generation call.

### Content source decision

- If the user provides content (recipes, story, etc.): use it directly.
- If the user asks the agent to create content: research and generate it.
  - For recipes: search for authentic recipes, adapt and write original versions.
  - For stories: generate an original narrative arc.
  - For guides: research the topic and structure the content.
- If mixed: use provided content and fill gaps with generated content.

### Visual references system — Recurring characters, objects & locations

#### Detection


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mechapizzai-lab/ebook-factory-gpt-image-2](https://github.com/mechapizzai-lab/ebook-factory-gpt-image-2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
