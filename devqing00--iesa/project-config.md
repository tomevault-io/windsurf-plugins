---
trigger: always_on
description: IESA is a departmental web platform built with Next.js 16 (App Router) and a FastAPI backend. It features JWT authentication, a dashboard with protected routes, and a **vibrant multi-color design system** using Tailwind CSS v4.
---

# Copilot Instructions for IESA Platform

## Project Overview

IESA is a departmental web platform built with Next.js 16 (App Router) and a FastAPI backend. It features JWT authentication, a dashboard with protected routes, and a **vibrant multi-color design system** using Tailwind CSS v4.

**Institution:** University of Ibadan (UI), Nigeria

---

## 🎨 DESIGN SYSTEM v3: Vibrant Academic

### Design Philosophy

Bold, vibrant, multi-color design inspired by modern card-based editorial layouts. Colorful bento grids, thick-bordered cards, playful but professional.

**Core Aesthetic Principles:**

- **Multi-Color Vibrancy** — Full palette: lime, lavender, coral, teal, sunny yellow
- **Thick & Chunky** — 4-8px borders, large radius (16–24px), bold button shapes
- **Hard Shadows** — Pure black/navy shadows (5-10px offset), never lime shadows
- **Bento Everything** — Asymmetric card grids with colored blocks, varied sizes
- **Typography-Led** — TBJ Endgraph black weight headlines, brush highlight accents
- **Playful-Professional** — Diamond sparkle decorators, fun but polished
- **Single Theme** — Light mode only, no dark mode support

---

## 🎨 TAILWIND CSS v4 THEME SYSTEM

### Theme Architecture

```css
/* 1. STATIC COLORS - in @theme, support opacity modifiers */
@theme {
  --color-lime: oklch(88% 0.2 128);
  --color-navy: oklch(15% 0.02 280);
}
/* Usage: bg-lime, text-navy, bg-lime/50, text-navy/60 */

/* 2. DYNAMIC COLORS - CSS variables that change with theme */
:root { --surface: #FAFAFE; }
.dark { --surface: oklch(15% 0.02 280); }

/* 3. DYNAMIC UTILITIES - Map CSS vars to Tailwind via @theme inline */
@theme inline { --color-surface: var(--surface); }
/* Usage: bg-surface (auto-switches with dark mode) */
```

### Color Usage Rules

**For static colors (don't change with theme):**
```jsx
// ✅ CORRECT - Uses @theme colors, supports opacity
<div className="bg-lime text-navy">
<div className="bg-lime/50 text-navy/70">
<div className="bg-lavender text-navy border-lime/30">

// ❌ WRONG - Raw hex or old color names
<div className="bg-[#C8F31D]">
<div className="bg-green-accent">
```

**Shadow & Border Rules:**
```jsx
// ✅ CORRECT - Pure black/navy shadows, navy borders
<div className="bg-snow border-[4px] border-navy shadow-[8px_8px_0_0_#000]">
<button className="bg-lime border-[4px] border-navy shadow-[5px_5px_0_0_#0F0F2D]">

// ❌ WRONG - Never use lime shadows or lime borders with black shadows
<div className="shadow-[8px_8px_0_0_#C8F31D]"> // Never lime shadow
<div className="border-lime shadow-[8px_8px_0_0_#000]"> // Conflicts
```

### Available Static Colors

| Color | Value | Usage |
|-------|-------|-------|
| `lime` | `oklch(88% 0.2 128)` | Primary accent, CTAs, active states |
| `lime-light` | `oklch(95% 0.08 128)` | Lime tint backgrounds |
| `lime-dark` | `oklch(75% 0.2 128)` | Hover/pressed states |
| `lavender` | `oklch(72% 0.12 295)` | Secondary: info, tags, decorative |
| `lavender-light` | `oklch(92% 0.05 295)` | Lavender tint backgrounds |
| `coral` | `oklch(68% 0.18 25)` | Alerts, deadlines, important |
| `coral-light` | `oklch(92% 0.06 25)` | Coral tint backgrounds |
| `teal` | `oklch(78% 0.14 175)` | Success, completion, positive |
| `teal-light` | `oklch(94% 0.05 175)` | Teal tint backgrounds |
| `sunny` | `oklch(85% 0.14 90)` | Warnings, stars, ratings |
| `sunny-light` | `oklch(96% 0.05 90)` | Yellow tint backgrounds |
| `navy` | `oklch(15% 0.02 280)` | Dark text, dark backgrounds |
| `navy-light` | `oklch(22% 0.02 280)` | Elevated dark surfaces |
| `navy-muted` | `oklch(35% 0.01 280)` | Secondary dark text |
| `slate` | `oklch(55% 0.01 280)` | Muted text, placeholders |
| `cloud` | `oklch(92% 0.005 280)` | Light borders, dividers |
| `ghost` | `oklch(97% 0.003 280)` | Off-white background |
| `snow` | `#ffffff` | White card surfaces |

### Shadow System Rules

**CRITICAL:** Follow these shadow rules strictly:

1. **Light backgrounds** (snow, ghost, lime-light, etc.) → Use pure black shadow: `shadow-[Xpx_Ypx_0_0_#000]`
2. **Lime buttons/elements** → Use navy shadow: `shadow-[Xpx_Ypx_0_0_#0F0F2D]`
3. **Dark backgrounds** (navy, navy-light) → Use lime shadow: `shadow-[Xpx_Ypx_0_0_#C8F31D]`
4. **NEVER use lime shadow on light backgrounds**
5. **Badges** (small labels like "Est. 2018") → No shadow at all

### Border System Rules

1. **Pair navy borders with black/navy shadows** → `border-navy shadow-[X_X_0_0_#000]`
2. **Never use lime borders when shadow is black** → This creates visual conflict
3. **Standard thickness:** 4-8px borders → `border-[4px]` or `border-[6px]`

---

## Typography System

**Font Families:**
- **Display Font:** `TBJ Endgraph` — Headlines, titles (weight: 900/black)
- **Body Font:** `TBJ Endgraph` — All text content (weights: 100, 300, 400, 500, 700)

**Font Weights:**
```css
.font-display    /* Black: 900 - Use for all headlines */
.font-bold      /* Bold: 700 - Use for emphasis */
.font-medium    /* Medium: 500 - Use for subheadings */
.font-normal    /* Regular: 400 - Use for body text */
.font-light     /* Light: 300 - Use sparingly */
.font-thin      /* Thin: 100 - Use for subtle text */
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devqing00/iesa](https://github.com/devqing00/iesa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
