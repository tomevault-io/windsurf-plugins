---
trigger: always_on
description: GlassKit is a pure CSS glassmorphism component library (v1.6.0) with 24 components, Dark & Light mode, design tokens, and BEM-like naming. Use this reference whenever generating HTML that uses GlassKit classes to ensure correct structure, nesting, modifiers, and token usage.
---


# GlassKit CSS – AI Component Reference

> **Purpose:** This document is an AI-optimized reference for generating correct GlassKit HTML markup.
> It replaces the need to parse `docs.html` and provides copy-paste-ready structures, rules, and composition patterns.

---

## 1. Setup & Boilerplate

### Including the Library

```html
<!-- CDN (recommended) -->
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@jungherz-de/glasskit@1.5/glasskit.min.css">

<!-- Local -->
<link rel="stylesheet" href="glasskit.css">

<!-- Optional: Load custom theme after base library -->
<link rel="stylesheet" href="theme-override.css">
```

### Minimal Template

```html
<!DOCTYPE html>
<html lang="en" data-theme="dark">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@jungherz-de/glasskit@1.5/glasskit.min.css">
</head>
<body>
  <div class="glass-bg">
    <!-- All content goes here -->
  </div>
</body>
</html>
```

### Naming Convention

- **Prefix:** All components use `glass-` (e.g. `glass-card`, `glass-btn`)
- **Utilities:** Use `gl-` (e.g. `gl-stack`, `gl-row`, `gl-mt-md`)
- **BEM logic:** `glass-component__element--modifier`
  - Element: `glass-card__text`, `glass-modal__header`
  - Modifier: `glass-btn--primary`, `glass-avatar--lg`
- **State classes:** `is-active`, `is-open`, `is-visible` (standalone, not BEM)

### Theming

The theme is controlled via `data-theme` on `<html>`:

```html
<!-- Dark Mode (default) -->
<html data-theme="dark">

<!-- Light Mode -->
<html data-theme="light">
```

Toggle theme via JavaScript:

```js
function toggleTheme() {
  const html = document.documentElement;
  const current = html.getAttribute('data-theme');
  html.setAttribute('data-theme', current === 'dark' ? 'light' : 'dark');
}
```

---

## 2. Design Tokens

All visual values are controlled via CSS Custom Properties. For custom theming, override them in a `theme-override.css`.

### Colors

| Token | Dark | Light | Usage |
|---|---|---|---|
| `--gl-color-primary` | `#f5a623` | `#e8852d` | Primary color, buttons, active elements |
| `--gl-color-primary-dark` | `#d4692a` | `#c96a1e` | Gradient end value |
| `--gl-color-primary-mid` | `#e07a24` | `#d97826` | Gradient midpoint |
| `--gl-color-text` | `#ffffff` | `#1a2a36` | Default text color |
| `--gl-color-text-muted` | `rgba(255,255,255,0.60)` | `rgba(26,42,54,0.55)` | Secondary text |
| `--gl-color-text-heading` | `#ffffff` | `#0f1f2a` | Headings |
| `--gl-color-success` | `#34c759` | `#28a745` | Success |
| `--gl-color-error` | `#ff3b30` | `#dc3545` | Error |
| `--gl-color-warning` | `#ffcc00` | `#e6a800` | Warning |

### Glass Surfaces

| Token | Dark | Usage |
|---|---|---|
| `--gl-surface-1` | `rgba(255,255,255, 0.08)` | Subtlest surface (status) |
| `--gl-surface-2` | `rgba(255,255,255, 0.10)` | Default (inputs, cards) |
| `--gl-surface-3` | `rgba(255,255,255, 0.14)` | Nav pills, badges |
| `--gl-surface-4` | `rgba(255,255,255, 0.16)` | Hover states |
| `--gl-surface-5` | `rgba(255,255,255, 0.22)` | Strong hover |
| `--gl-surface-milk` | `rgba(255,255,255, 0.55)` | Milky |
| `--gl-surface-milk-strong` | `rgba(255,255,255, 0.75)` | Secondary button |
| `--gl-surface-overlay` | `rgba(0,0,0, 0.50)` | Modal overlay |

### Borders

| Token | Value |
|---|---|
| `--gl-border-subtle` | `rgba(255,255,255, 0.18)` |
| `--gl-border-medium` | `rgba(255,255,255, 0.30)` |
| `--gl-border-strong` | `rgba(255,255,255, 0.40)` |
| `--gl-border-milk` | `rgba(255,255,255, 0.60)` |
| `--gl-border-warm` | `rgba(255,200,100, 0.35)` |
| `--gl-border-focus` | `rgba(245,166,35, 0.60)` |

### Blur

| Token | Value |
|---|---|
| `--gl-blur` | `24px` (default) |
| `--gl-blur-light` | `16px` |
| `--gl-blur-soft` | `12px` |
| `--gl-blur-heavy` | `40px` |

### Radii

| Token | Value | Usage |
|---|---|---|
| `--gl-radius-xs` | `8px` | Small elements |
| `--gl-radius-sm` | `12px` | Badges, small containers |
| `--gl-radius-input` | `14px` | Inputs, textareas |
| `--gl-radius-btn` | `16px` | Buttons |
| `--gl-radius-card` | `24px` | Cards |
| `--gl-radius-full` | `9999px` | Fully rounded |
| `--gl-radius-pill` | `50%` | Circle shape |

### Spacing

| Token | Value |
|---|---|
| `--gl-space-2xs` | `4px` |
| `--gl-space-xs` | `8px` |
| `--gl-space-sm` | `12px` |
| `--gl-space-md` | `16px` |
| `--gl-space-lg` | `20px` |
| `--gl-space-xl` | `24px` |
| `--gl-space-2xl` | `32px` |
| `--gl-space-3xl` | `40px` |
| `--gl-space-4xl` | `56px` |

### Shadows

| Token | Usage |
|---|---|
| `--gl-shadow-card` | Cards |
| `--gl-shadow-btn` | Default buttons |
| `--gl-shadow-btn-primary` | Primary button (orange glow) |
| `--gl-shadow-glow` | Glow effect |
| `--gl-shadow-modal` | Modal dialog |
| `--gl-shadow-toast` | Toast notifications |
| `--gl-shadow-focus` | Focus ring |

### Typography

| Token | Value |
|---|---|
| `--gl-font-size-xs` | `13px` |
| `--gl-font-size-sm` | `14px` |
| `--gl-font-size-base` | `15px` |
| `--gl-font-size-btn` | `16px` |
| `--gl-font-size-lg` | `18px` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JUNGHERZ/GlassKit](https://github.com/JUNGHERZ/GlassKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
