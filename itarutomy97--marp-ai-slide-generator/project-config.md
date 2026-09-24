---
trigger: always_on
description: This file provides guidance to Gemini when working with code in this repository.
---

# GEMINI.md

This file provides guidance to Gemini when working with code in this repository.

## Project Overview

This is a Marp (Markdown Presentation) template repository for creating professional presentations. Marp converts Markdown files into HTML, PDF, or PPTX presentations with custom styling.

## Flexible Task Handling

This directory may be used for tasks beyond slide creation. Respond flexibly based on user instructions:

### Supported Task Types

1. **Slide Creation**: Creating presentation materials using Marp (primary purpose)
2. **Planning Documents**: Creating proposals and outlines for presentations
3. **Research**: Gathering information needed for slide creation
4. **Review**: Reviewing and suggesting improvements to existing slides
5. **Other Documents**: Meeting notes, reports, and related documentation

### Decision Criteria

- Keywords like "slide", "presentation", "Marp" → Slide creation task
- Keywords like "plan", "outline", "idea", "proposal" → Planning document task
- If unclear, ask the user before starting

## Slide Creation Workflow (Recommended)

When creating slides, use the **generator (JSON→Markdown conversion) workflow** as the standard approach.

### Recommended Workflow: JSON + Generator

1. **Create slide structure in JSON format**
   - Use existing templates in `generator/templates/`
   - Define `type` and `data` for each slide

2. **Generate Markdown with Generator**
   ```bash
   node generator/md_build.js <input.json> [output.md]
   ```

3. **Fine-tune Markdown if needed**
   - Edit the generated Markdown directly for adjustments

4. **Convert to HTML/PDF/PPTX**

### Why Use the Generator

- **Reusability**: Leverage templates in `generator/templates/`
- **Consistency**: Maintain unified design system
- **Efficiency**: Generate large numbers of slides efficiently
- **Maintainability**: Update slides by modifying JSON only

### Available Templates

Over 30 templates available in `generator/templates/`:

See `generator/README.md` for details.

## Adding New Components/Templates

Follow these steps and rules when adding new slide layouts.

### Files to Create

| File | Location | Purpose |
|------|----------|---------|
| HBS Template | `generator/templates/<name>.hbs` | Generator template |
| Component | `generator/templates/<name>.md` | Preview/documentation |

### Component (.md) Requirements

1. **Include Marp frontmatter** (for preview capability)
   ```markdown
   ---
   marp: true
   theme: deskrex
   size: 16:9
   paginate: true
   header: "Header Text"
   footer: "@your-handle"
   ---
   ```

2. **Include JSON structure example in comments**
   ```markdown
   <!--
     Component name and description

     JSON structure example:
     {
       "type": "template-name",
       "data": { ... }
     }
   -->
   ```

3. **Separate multiple patterns with `---`**
   - Show variations as multiple slides

### Naming Conventions

- **Use generic names**
  - Good: `image-grid`
  - Bad: `client-logos-grid` (too specific)
- **Use kebab-case**: `three-step-process`, `image-grid`

### Directory Structure Rules

- **Place directly under `generator/templates/`** (no subdirectories)
  - Good: `generator/templates/image-grid.md`
  - Bad: `generator/templates/layouts/image-grid.md`

### Post-Addition Tasks

1. **Update `generator/README.md`**: Add to template list
2. **Verify**: Preview component display in Marp

### JSON Structure Example

```json
{
  "meta": {
    "theme": "deskrex",
    "header": "Presentation Title",
    "footer": "@your-handle"
  },
  "slides": [
    {
      "type": "title-cover",
      "data": {
        "mainTitle": "Main Title",
        "subtitle": "Subtitle"
      }
    },
    {
      "type": "section-divider",
      "data": {
        "title": "Chapter 1",
        "subtitle": "Section Name"
      }
    }
  ]
}
```

## Common Commands

### Generate Slides with Generator (Recommended)

```bash
# Generate Markdown from JSON (no timestamp, default)
node generator/md_build.js <input.json>
# → Generates input.marp.md

# Output with timestamp
node generator/md_build.js <input.json> --timestamp
# → Generates input_20251215_120000.md

# Specify output path
node generator/md_build.js slides.json output/presentation.md
```

### Split JSON Management and Merging

Large presentations can be split into sections for management.

**File naming convention**: `part_XX_section-name.json` (XX is a number)

```
projects/my-presentation/
├── part_00_intro.json           # Merge target
├── part_01_about.json           # Merge target
├── part_02_main.json            # Merge target
├── part_03_closing.json         # Merge target
├── config.json                  # Excluded (doesn't start with part_)
└── slides.json                  # ← Merge output destination
```

**Merge commands**:
```bash
# Merge part_XX_*.json and generate Markdown
node generator/merge-json.js <project-dir> --build

# Specify output filename
node generator/merge-json.js <project-dir> --output <name> --build

# Example
node generator/merge-json.js ./projects/my-presentation --output slides --build
```

**Benefits**:
- Edit sections independently
- Easier to manage large presentations
- Auto-sorted by number and merged

### Presentation Conversion

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [itarutomy97/marp-ai-slide-generator](https://github.com/itarutomy97/marp-ai-slide-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
