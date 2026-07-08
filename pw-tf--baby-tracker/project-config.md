---
trigger: always_on
description: This file is the source of truth for how Claude Code should approach all web projects in this workspace. It covers design system usage, coding conventions, and Supabase backend rules. Read it fully before beginning any task.
---

# CLAUDE.md — Web Development Baseline

This file is the source of truth for how Claude Code should approach all web projects in this workspace. It covers design system usage, coding conventions, and Supabase backend rules. Read it fully before beginning any task.

---

## 1. DESIGN SYSTEM

All projects reference the award-winning design system documented below. Do not invent ad-hoc colours, spacing, or typography — use the variables and patterns defined here.

### 1.1 How to Apply the Design System

- **Existing projects:** Check whether a palette has already been established (look for `:root` CSS variables). If one exists, extend it using the master template variables. Do not replace an established palette unless explicitly instructed.
- **New projects:** Ask which palette to use before writing any styles. Present the six options from section 1.3 with a one-line description each.
- **Always:** Implement using CSS custom properties (`--color-primary`, etc.) — never hardcode hex values into component styles.

---

### 1.2 CSS Custom Properties Master Template

Paste into the root stylesheet and override palette variables per project:

```css
/* ============================================
   DESIGN SYSTEM — Root Variables
   ============================================ */

:root {
  /* — PALETTE (swap these per project) — */
  --color-bg:         #F5F0E8;
  --color-surface:    #FFFFFF;
  --color-primary:    #49C5B6;
  --color-secondary:  #DF6C4F;
  --color-accent:     #ECD06F;
  --color-text:       #111111;
  --color-text-muted: #6B6B6B;
  --color-border:     rgba(0,0,0,0.1);

  /* — TYPOGRAPHY — */
  --font-display: 'DM Sans', sans-serif;
  --font-body:    'DM Sans', sans-serif;

  --text-xs:   0.75rem;
  --text-sm:   0.875rem;
  --text-base: 1rem;
  --text-lg:   1.25rem;
  --text-xl:   1.75rem;
  --text-2xl:  2.5rem;
  --text-3xl:  clamp(2.5rem, 5vw, 4rem);
  --text-hero: clamp(4rem, 12vw, 10rem);

  --weight-regular: 400;
  --weight-bold:    700;
  --weight-black:   900;

  --leading-tight:  0.9;
  --leading-snug:   1.2;
  --leading-normal: 1.5;
  --leading-loose:  1.75;

  --tracking-tight:  -0.04em;
  --tracking-normal:  0;
  --tracking-wide:    0.08em;

  /* — SPACING — */
  --space-xs:  0.5rem;
  --space-sm:  1rem;
  --space-md:  2rem;
  --space-lg:  4rem;
  --space-xl:  8rem;
  --space-2xl: 14rem;

  /* — SHAPE — */
  --radius-sm:   4px;
  --radius-md:   12px;
  --radius-lg:   24px;
  --radius-full: 9999px;

  /* — ANIMATION — */
  --ease-out-expo: cubic-bezier(0.16, 1, 0.3, 1);
  --ease-in-out:   cubic-bezier(0.4, 0, 0.2, 1);
  --duration-fast: 0.2s;
  --duration-base: 0.4s;
  --duration-slow: 0.7s;

  /* — SHADOWS — */
  --shadow-sm: 0 2px 8px rgba(0,0,0,0.06);
  --shadow-md: 0 8px 24px rgba(0,0,0,0.1);
  --shadow-lg: 0 20px 60px rgba(0,0,0,0.15);
}
```

---

### 1.3 Colour Palettes

Six palettes are available. Each is pre-built as a `:root` override block — paste over the palette section of the master template.

---

#### Palette A — "Terracotta Studio"
**Mood:** Bold, minimal, warm. Best for food & drink, lifestyle, portfolios.

```css
:root {
  --color-primary:    #DF6C4F;
  --color-bg:         #F5F0EB;
  --color-surface:    #FFFFFF;
  --color-text:       #111111;
  --color-accent:     #1A1A1A;
}
```

**Key trait:** Single saturated terracotta accent on warm off-white. Use sparingly for CTAs only.

---

#### Palette B — "Teal & Bloom"
**Mood:** Fresh, playful, modern. Best for e-commerce, wellness, app landing pages.

```css
:root {
  --color-primary:    #49C5B6;
  --color-secondary:  #FF9398;
  --color-accent:     #ECD06F;
  --color-bg:         #FAFAFA;
  --color-text:       #1C1C2E;
  --color-surface:    #FFFFFF;
}
```

**Key trait:** Scroll-triggered background colour transitions — each section owns one of the three accent colours.

---

#### Palette C — "Electric Neon"
**Mood:** High-energy, tech, disruptive. Best for startups, dev tools, fintech, Web3.

```css
:root {
  --color-primary:    #C8FF00;
  --color-bg:         #0D1230;
  --color-surface:    #161A3A;
  --color-text:       #FFFFFF;
  --color-accent:     #3D6EFF;
  --color-highlight:  #FF4D3D;
}
```

**Key trait:** Dark-first. Electric lime dominates the navbar/logo. CTAs in bright blue stand out strongly against the dark field.

---

#### Palette D — "Carnival Burst"
**Mood:** Vibrant, cultural, editorial. Best for music, events, entertainment, media.

```css
:root {
  --color-red:        #D4421A;
  --color-yellow:     #F5E642;
  --color-green:      #38C98C;
  --color-purple:     #9B88CC;
  --color-bg:         #111111;
  --color-text:       #111111;
  --color-text-inv:   #FFFFFF;
}
```

**Key trait:** Each section uses a different palette colour as a full-bleed background. Hover effects swap tile colours. Bold condensed typography fills each block.

---

#### Palette E — "Editorial Cream"
**Mood:** Sophisticated, luxury, French minimalism. Best for agencies, luxury brands, editorial, law, architecture.

```css
:root {
  --color-primary:    #1B1F8A;
  --color-bg:         #F5F0E8;
  --color-surface:    #EDE8DC;
  --color-text:       #1A1A1A;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pw-tf/baby-tracker](https://github.com/pw-tf/baby-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
