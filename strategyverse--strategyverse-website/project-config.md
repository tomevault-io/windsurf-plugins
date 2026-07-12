---
trigger: always_on
description: ﻿# Strategyverse Website — Project Reference
---

﻿# Strategyverse Website — Project Reference

## Standing Rules

- **All new content and changes must be AEO compliant.** This applies to every new blog post, page edit, schema update, and structural change. See the AEO Compliance Checklist section below for what this means in practice.

---

## Overview

Strategyverse Consulting is a **Strategic Public Relations** company based in Noida, India. This is a static HTML/CSS/JS website (no build tools, no frameworks) hosted on **GitHub Pages** at:

**https://strategyverse.in** (custom domain, DNS pointed from GitHub Pages)

Repository: `https://github.com/StrategyVerse/StrategyVerse-website`

---

## Brand Identity

### Colours

| Token            | Hex       | Usage                                    |
| ---------------- | --------- | ---------------------------------------- |
| `--blue`         | `#377FCC` | Primary brand colour, buttons, links     |
| `--blue-dark`    | `#2a6ab3` | Hover / active states                    |
| `--blue-deeper`  | `#1a3a5c` | Hero gradient, deep accents              |
| `--amber`        | `#F5A623` | Secondary accent, section labels, checks |
| `--amber-dark`   | `#d98e1a` | Amber hover states                       |
| `--dark`         | `#1a1a2e` | Heading text, dark backgrounds           |
| `--dark-light`   | `#16213e` | Hero gradient end                        |
| `--gray-100`     | `#f7f8fa` | Light section backgrounds                |
| `--gray-200`     | `#e9ecef` | Borders, dividers                        |
| `--gray-400`     | `#adb5bd` | Muted text                               |
| `--gray-600`     | `#6c757d` | Body text, subtitles                     |
| `--gray-800`     | `#343a40` | Default body text colour                 |
| `--white`        | `#ffffff` | Backgrounds, text on dark                |

### Fonts

- **Headings:** `Playfair Display` (weights 600, 700, 800) — serif, elegant
- **Body:** `Inter` (weights 400, 500, 600, 700) — clean sans-serif
- Loaded from Google Fonts on every page

### Typography Scale

```
h1  → clamp(2.2rem, 5vw, 3.5rem)
h2  → clamp(1.8rem, 4vw, 2.8rem)
h3  → clamp(1.2rem, 2.5vw, 1.5rem)
body → 16px, line-height 1.7
section-label → 0.85rem, uppercase, 3px letter-spacing, amber colour
```

### Shadows

```
--shadow-sm: 0 2px 8px rgba(0,0,0,0.08)
--shadow-md: 0 4px 20px rgba(0,0,0,0.12)
--shadow-lg: 0 8px 40px rgba(0,0,0,0.16)
```

### Buttons

- `.btn-primary` — Blue (#377FCC), white text, pill shape (border-radius 50px)
- `.btn-outline` — Transparent, white border, used on dark backgrounds
- `.btn-amber` — Amber (#F5A623), dark text
- `.btn-arrow` — Appends animated `→` that translates right on hover

---

## Blog Cover Illustration Style (Signature House Style)

All blog cover images use a **signature illustration style** unique to Strategyverse — not stock photos. Every cover must follow this system so the Insights grid reads as one unmistakable visual language.

### The fixed signature (constant on every cover)

- **Medium:** highly detailed scratchboard / wood-engraving illustration in the vintage-etching tradition (Gustave Doré feel) — dense cross-hatching, flowing parallel linework, extreme contrast.
- **Palette:** monochrome duotone — deep midnight **navy ink (`#1a1a2e`)** on a **warm off-white paper** background (with a thin paper border), never pure black.
- **The one-amber rule:** exactly **one element glows warm amber (`#F5A623`)** and it is the ONLY colour in the entire frame — the focal "signal/insight." Everything else is navy linework on off-white.
- **Composition:** a lone figure in a vast, dramatic landscape; epic scale; contemplative, aspirational mood; clean negative space. No text or lettering baked into the image (the page renders the H1 separately).

### What varies: topic-matched metaphor per post

Each article gets its own conceptual scene chosen to fit the topic, drawn in the fixed style above. Examples used: navigation (boat through a maelstrom toward an amber dawn = strategy/turbulence), voice (figure broadcasting through a horn = communications), rising (staircase of books to an amber sun = thought leadership), beacon on a cliff (signal through noise).

### Production

- **Tool/model:** image-generation MCP, model **`flux-2-max`**, aspect ratio **16:9**, PNG/JPG. ~14 credits per image — generate deliberately, don't regenerate idly.
- **Prompt template:** `Highly detailed scratchboard and wood-engraving illustration in the style of vintage etching and Gustave Doré — dense intricate cross-hatching and fine flowing parallel linework, extreme high contrast, dramatic and cinematic. Monochrome duotone in deep midnight navy ink (color #1a1a2e) on a warm off-white paper background with a thin paper border, NOT pure black. The scene: [TOPIC METAPHOR]. One element — [the amber element] — glows warm amber (color #F5A623) and it is the ONLY color in the entire image; everything else is navy engraved linework on off-white. Epic sense of scale, contemplative and aspirational mood, editorial illustration. No text, no lettering.`
- **Storage:** save each cover to `images/blog/<slug>.jpg` and reference it locally (replaces the old Pexels URLs).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [StrategyVerse/StrategyVerse-website](https://github.com/StrategyVerse/StrategyVerse-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
