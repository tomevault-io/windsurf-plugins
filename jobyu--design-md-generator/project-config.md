---
trigger: always_on
description: Generate a comprehensive DESIGN.md design system document for web and mobile app projects, from project context, website URL, design screenshot, or by guiding a human step-by-step. Use this skill whenever the user asks for creating a design system, generating DESIGN.md, analyzing a website's visual design, extracting design elements from a screenshot or mockup, or producing visual/interaction specs for web or mobile apps — even if they don't explicitly use the word "DESIGN.md".
---


## Step 1: Input Detection & Mode Routing

Automatically detect the user's input type and route to the appropriate mode. **Do not ask the user which mode they want if the input type is unambiguous.**

### Input Type Decision Tree

```
User Input
├── URL (starts with http:// or https://)
│   └── → URL Analysis Mode (see "URL Analysis Mode" below)
├── Image file (uploaded .png/.jpg/.jpeg/.webp/.gif or provided path)
│   └── → Image Analysis Mode (see "Image Analysis Mode" below)
├── Existing project files (README.md, package.json, code files detected)
│   └── → Autonomous Mode (see "Autonomous Mode" below)
└── Text description only / no specific input
    └── → Guided Mode (see "Guided Mode" below)
```

### Ambiguous Input Handling

If the user provides **multiple types** (e.g., a URL + a screenshot), prioritize in this order:
1. **Image** (most specific visual source)
2. **URL** (live, interactive source)
3. **Project files** (contextual source)

Then ask:

> "I can work with multiple inputs here. Should I prioritize the [image/URL] for visual extraction, or the project files for context?"

---

## Mode Summaries

### Autonomous Mode

Analyze existing project files (README, manifest, styles) to infer design direction. Pick a design personality from the archetypes below based on product type and tech stack. **Avoid "AI flavor"** — do not default to Inter + `#3B82F6`. See "Autonomous Mode" section below for the full procedure.

### URL Analysis Mode

Extract colors, typography, components, layout, shadows, and border radius from a live website using WebFetch. Infer product type, audience, and emotional metaphor from the visual traits. Mark unextractable fields (deep narrative, "unreasonable choice", brand taboos) as `[MISSING]`. See "URL Analysis Mode" section below for the full extraction checklist.

### Image Analysis Mode

Analyze a design screenshot or mockup image. Extract color palette, font categories, component styles, layout density, depth/elevation, and overall atmosphere. Mark unidentifiable fields (exact font names, precise pixel values, transition timing) as `[MISSING]`. See "Image Analysis Mode" section below for the full visual analysis checklist.

### Guided Mode

Two sub-modes:
- **Full Interview** (standalone): Ask 8 structured questions one at a time (product, metaphor, unreasonable choice, platform, color, typography, density, motion).
- **Gap-Filling** (post-analysis): After URL/Image analysis, ask only the missing fields. Do not repeat already-extracted information.

See "Guided Mode" section below for the full question bank and gap-filling templates.

---

## Step 2: Data Sufficiency Check & Auto-Guided Gap-Filling

### Sufficiency Checklist

Before generating the final DESIGN.md, verify that all critical fields are filled (not placeholder, not `[MISSING]`):

| Priority | Check Item | Threshold |
|----------|------------|-----------|
| **P0** | Product name | Must have |
| **P0** | At least 1 primary color with hex | Must have |
| **P0** | Background color + text color | Must have |
| **P0** | Font family (heading + body) | Must have |
| **P0** | Emotional metaphor (2-3 sentences) | Must have |
| **P1** | "Unreasonable but right" choice | Must have |
| **P1** | Button style (padding, radius, colors) | Must have |
| **P1** | Card/container style | Must have |
| **P2** | Spacing system | Can infer |
| **P2** | Shadow system | Can infer |
| **P2** | Border radius scale | Can infer |
| **P2** | Breakpoints | Can infer |
| **P2** | Semantic colors | Can infer |
| **P2** | Do's and Don'ts | Can infer from metaphor |

### Decision Matrix

| Missing P0 | Missing P1 | Missing P2 | Action |
|------------|------------|------------|--------|
| 0 | 0 | 0-3 | **Generate immediately** with inferred P2 values |
| 0 | 1-2 | any | **Generate draft** + ask 1-2 targeted questions |
| 0 | 3+ | any | **Enter Gap-Filling** (see "Guided Mode" below) |
| 1+ | any | any | **Enter Gap-Filling**, must resolve P0 |

### Inference Rules for Missing Fields

When auto-generating values for insufficient data, follow these rules:

**If spacing system is missing:**
- Default to 4px base unit with scale: 4, 8, 12, 16, 24, 32, 48, 64, 96
- Adjust for density: dense products use 4/8/12/16; airy products use 8/16/24/32/48

**If shadow system is missing:**
- Vercel/Linear style (minimal shadows, borders instead) → use subtle 1px borders
- Material/Figma style → use layered shadows: `0 1px 3px rgba(0,0,0,0.1)`, `0 4px 12px rgba(0,0,0,0.08)`, `0 12px 32px rgba(0,0,0,0.12)`
- Apple/Notion style → use very subtle shadows or none

**If border radius is missing:**
- Engineer/developer tool → 4-6px (sharp, precise)
- Consumer/warm brand → 8-12px (friendly)
- Playful/creative → 16-24px (soft)
- Medical/finance → 2-4px (serious)

**If breakpoints are missing:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JobYu/design-md-generator](https://github.com/JobYu/design-md-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
