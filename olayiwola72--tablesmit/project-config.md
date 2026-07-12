---
trigger: always_on
description: > For AI Coding Agents (Codex, Claude Code, etc.)
---

# Tablesmit — Brand Identity & Engineering Implementation Guide
> For AI Coding Agents (Codex, Claude Code, etc.)
> Brand + Positioning + Architecture + TDD | Tailwind CSS Edition
> Status: Authoritative. Do not deviate without explicit instruction.
> Current Release: v1.3.0

---

## 0. The North Star

**Tablesmit** is a minimalist table builder for analytical writing.

It exists for writers, analysts, researchers, and technical thinkers who need clean
structured tables with full control over headers, formatting, and export —
without the noise of a spreadsheet, the complexity of a database, or the
aesthetic overwhelm of a design tool.

> **Tagline:** *Tables, your way.*
> **Subtext (hero):** *A minimalist table builder for analytical writing — with full control over headers, formatting, and export.*
> **About line:** *Tablesmit was created by a writer who needed more control than basic table generators provided. Built for people who think in structure and publish with precision.*

### Positioning Statement
```
For:     Writers, analysts, researchers, and technical thinkers
Who:     Need clean, structured tables with customization control
Tablesmit is: A minimalist table builder for analytical writing
That:    Gives full control over headers, formatting, and export
Unlike:  Generic table tools that are either too rigid or too complex
```

### What Tablesmit Is Not
```
Not a spreadsheet.
Not a database.
Not a Notion competitor.
Not a design-heavy tool.

Tablesmit is a structured writing tool.
```
Every product decision must be filtered through this. If a feature makes
Tablesmit feel like any of the above — reconsider it.

### Tone of Voice
| Dimension   | Direction                                                        |
|-------------|------------------------------------------------------------------|
| Personality | Friendly productivity. Calm. Competent. Minimal.                 |
| Writing     | Short sentences. Active voice. No marketing fluff.               |
| UI copy     | Direct. Label the action. No clever wordplay in functional UI.   |
| Error msgs  | Clear cause + clear fix. Never blame the user.                   |
| Empty states| Invite action. Do not describe what is missing.                  |

### Emotional Design Goal
Every UI decision must make the user feel:

> **Clarity. Control. Calm confidence.**

If a design choice introduces visual noise, decision fatigue, or uncertainty —
remove it. When in doubt: simplify.

---

## 1. Brand Name & Identity

| Field          | Value                                                                              |
|----------------|------------------------------------------------------------------------------------|
| Product Name   | **Tablesmit**                                                                      |
| Domain         | tablesmit.com (confirmed, ~$11/year)                                               |
| Origin         | Table + Smith. A smith crafts with precision — wordsmith, goldsmith, tablesmith.   |
|                | The missing "h" is intentional — own it as the brand spelling (cf. Tumblr, Flickr)|
| Pronunciation  | TAY-bul-smit                                                                       |
| SEO note       | Cover "tablesmith" in meta description and GitHub — users will search both spellings|
| Personality    | Calm. Competent. Minimal. Friendly but never chatty.                               |
| Open Source    | Yes — GitHub link in secondary CTA + Sponsor button                                |
| What it is NOT | Loud, cluttered, feature-heavy, or visually expressive.                            |

---

## 2. Logo

The only logo is the **T-form** — three rectangles forming a table with a header row.
No outlines, no strokes. Pure filled shapes only.
Rendered as a React SVG component (`src/components/ui/Logo/Logo.tsx`), not an external image file.

### Concept

Three rectangles. Full opacity on top (the header row), fading opacity below
(two data columns). Reads as a table with a header from 16px up.
The decreasing opacity from left column to right subtly implies "more columns beyond."
The shape is also a T — a quiet reference to the brand name.

### 2A. Full Logo (Icon + Wordmark)

**Light background:**
```svg
<svg width="220" height="48" viewBox="0 0 220 48" fill="none"
     xmlns="http://www.w3.org/2000/svg">
  <g transform="translate(4,8)">
    <rect x="0" y="0" width="30" height="10" rx="4" fill="#1E40AF"/>
    <rect x="0" y="13" width="13" height="15" rx="3" fill="#1E40AF" opacity="0.28"/>
    <rect x="17" y="13" width="13" height="15" rx="3" fill="#1E40AF" opacity="0.14"/>
  </g>
  <text x="46" y="30"
        font-family="Inter, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif"
        font-size="22" font-weight="600" letter-spacing="-0.5"
        fill="#1E293B">Tablesmit</text>
</svg>
```

**Dark background:**
```svg
<svg width="220" height="48" viewBox="0 0 220 48" fill="none"
     xmlns="http://www.w3.org/2000/svg">
  <g transform="translate(4,8)">
    <rect x="0" y="0" width="30" height="10" rx="4" fill="#60A5FA"/>
    <rect x="0" y="13" width="13" height="15" rx="3" fill="#60A5FA" opacity="0.35"/>
    <rect x="17" y="13" width="13" height="15" rx="3" fill="#60A5FA" opacity="0.18"/>
  </g>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Olayiwola72/tablesmit](https://github.com/Olayiwola72/tablesmit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
