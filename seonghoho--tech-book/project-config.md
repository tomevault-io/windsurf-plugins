---
trigger: always_on
description: This document describes the design system, component patterns, and conventions used in this codebase to help integrate Figma designs accurately.
---

# CLAUDE.md — Figma Design Integration Guide

This document describes the design system, component patterns, and conventions used in this codebase to help integrate Figma designs accurately.

---

## Stack Overview

| Layer | Technology |
|---|---|
| Framework | Next.js 16 (App Router), React 19, TypeScript 5 |
| Styling | Tailwind CSS v3 + custom CSS variables |
| 3D / Canvas | Three.js, @react-three/fiber, @react-three/drei |
| Animation | GSAP, @react-spring/web, @use-gesture/react |
| Content | Markdown via gray-matter + remark/rehype pipeline |
| Deployment | Vercel (Analytics + Speed Insights) |
| Dev server | `npm run dev` — Next.js with Turbopack on port 5174 |
| Build | `npm run build` (runs `build:posts` then `next build`) |

---

## 1. Design Tokens

### CSS Variables (Source of Truth)

All color tokens are defined as CSS custom properties in [`src/styles/globals.css`](src/styles/globals.css).

**Light mode (`:root`):**
```css
--color-bg: #f4f7fb;
--color-bg-muted: #edf2f7;
--color-surface: #ffffff;
--color-surface-elevated: #f8fbfd;
--color-surface-muted: #e9eff6;
--color-surface-tint: #f2fbf8;
--color-text-primary: #0f172a;
--color-text-secondary: #475569;
--color-text-muted: #64748b;
--color-border: #d8e0ea;
--color-border-strong: #c4ceda;
--color-accent: #0f766e;         /* teal — primary interactive color */
--color-accent-hover: #115e59;
--color-accent-soft: rgba(15, 118, 110, 0.12);
--color-link: #0f766e;
--color-code: #eff4f9;
--shadow-soft: 0 18px 40px rgba(15, 23, 42, 0.06);
--shadow-medium: 0 24px 60px rgba(15, 23, 42, 0.08);
```

**Dark mode (`html.dark`):**
```css
--color-bg: #0b1320;
--color-surface: #111b2a;
--color-surface-elevated: #162234;
--color-text-primary: #e5edf6;
--color-text-secondary: #c5d0de;
--color-text-muted: #8fa2b7;
--color-accent: #58c9b9;         /* lighter teal in dark mode */
--color-accent-hover: #7edfd1;
--color-link: #7edfd1;
--color-code: #0f1727;
--shadow-soft: 0 18px 48px rgba(2, 8, 23, 0.28);
--shadow-medium: 0 28px 70px rgba(2, 8, 23, 0.34);
```

### Tailwind Token Extensions ([`tailwind.config.mjs`](tailwind.config.mjs))

```js
colors: {
  border: "var(--color-border)",   // shorthand Tailwind alias
  gray: "#868e96",
  dark: "#0F0F0F",
  bright: "#FFFFFF",
},
fontFamily: {
  press: ['"PressStart2P"', "cursive"],  // pixel/retro font for games UI
},
boxShadow: {
  "bg-glow": "inset 0 0 30px rgba(0,0,0,0.1)",
},
```

**Dark mode strategy:** `darkMode: "class"` — toggled by adding/removing `dark` class on `<html>`.

### How to reference tokens in new components

Always use CSS variables via the `[color:var(--color-*)]` Tailwind arbitrary-value syntax:

```tsx
// Correct
<p className="text-[color:var(--color-text-secondary)]">...</p>
<div className="border-[color:var(--color-border)]">...</div>

// Or via the 'border' Tailwind alias
<div className="border-border">...</div>
```

---

## 2. Typography

### Fonts

Defined in [`src/styles/fonts.css`](src/styles/fonts.css) as `@font-face` rules, served from `/public/fonts/`.

| Font | Weights | Usage |
|---|---|---|
| Pretendard | 400, 500, 600, 700 | All body/UI text (Korean + Latin) |
| PressStart2P | 400 | Games UI only (`font-press` Tailwind class) |

**Body font stack:**
```css
font-family: "Pretendard", "Apple SD Gothic Neo", "Noto Sans KR", "Malgun Gothic", sans-serif;
```

### Typography Scale Patterns

Used directly in prose content via `@tailwindcss/typography` plugin. The prose styles use CSS variables for colors so they respond to the theme automatically. Max prose width: `72ch`.

---

## 3. Utility Component Classes

Reusable semantic classes are defined in the `@layer components` block of [`src/styles/globals.css`](src/styles/globals.css). Always use these instead of recreating the patterns inline.

### Surfaces / Cards

```tsx
// Standard card — rounded-[28px], border, bg-surface, shadow-soft
<div className="surface-panel p-5">...</div>

// Elevated card — rounded-[32px], bg-surface-elevated, shadow-medium
<div className="surface-panel-strong p-6">...</div>

// Subtle inset — rounded-[12px], border, bg-surface-elevated
<div className="surface-subtle px-4 py-3">...</div>

// About/portfolio card (same as surface-panel)
<div className="about-card p-5">...</div>

// Interactive card with hover lift + color shift
<div className="about-card-interactive px-5 py-4">...</div>
```

### Text Utilities

```tsx
<p className="eyebrow-label">SECTION LABEL</p>         // uppercase, tracking-[0.24em], accent color
<h2 className="section-title">Title</h2>               // text-2xl sm:text-3xl, font-semibold
<p className="body-copy">Body text</p>                  // text-sm sm:text-base, leading-7, text-secondary
<p className="muted-copy">Smaller note</p>              // text-sm leading-6, text-muted
```

### Buttons

```tsx
<button className="button-primary">Primary CTA</button>    // filled, rounded-full, inverts on hover to accent
<button className="button-secondary">Secondary</button>    // outlined, rounded-full
```

### Interactive Elements

```tsx
<span className="tag-chip">Tag</span>        // pill chip, border, bg-surface-elevated

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/seonghoho) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
