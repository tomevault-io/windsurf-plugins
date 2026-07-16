---
trigger: always_on
description: This Skill is a pure instruction set — it does not make network requests, call APIs, or execute code.
---

# Apple Design Skill — CLAUDE.md

> **Self-contained skill file for Claude Code.** This file includes all design tokens, typography rules, copywriting patterns, image curation strategies, and layout templates needed to generate Apple-style frontend UI code. No external file references required.

---

## Identity & Purpose

You are an AI assistant enhanced with the **Apple Design Skill**. Generate frontend UI code that faithfully reproduces the design aesthetic of apple.com — generous whitespace, typographic precision, restrained color palettes, and relentless clarity.

**Language support:** English (en), Simplified Chinese (zh-CN), Traditional Chinese (zh-TW), Japanese (ja), Korean (ko). Detect the user's language and apply the corresponding font stack and copywriting rules automatically.

---

## Core Design Principles

1. **Simplicity over decoration.** Remove every element that does not serve a clear purpose.
2. **Whitespace is a feature.** 80–120px between sections gives content room to breathe.
3. **Typography-first hierarchy.** Large bold headlines (48–80px) anchor each section; subheadings and body step down in size and weight.
4. **Restrained color palette.** #FFFFFF, #F5F5F7, #1D1D1F for backgrounds. Accent blue (#0066CC) sparingly for interactive elements.
5. **Precision in detail.** Rounded corners (12–20px), multi-layer shadows, smooth transitions.
6. **Content-centered layout.** Constrain content to 980–1200px, centered. Full-bleed imagery may extend beyond.
7. **Responsive by default.** Three breakpoints: 734px, 1068px, 1440px.

---

## Design Tokens

All visual variables as CSS custom properties. Copy the `:root` blocks into your `<style>` tag. Never hard-code raw values.

### Colors

```css
:root {
  --apple-bg-white: #FFFFFF;
  --apple-bg-dark: #1D1D1F;
  --apple-bg-light-gray: #F5F5F7;
  --apple-bg-elevated: #FBFBFD;

  --apple-text-primary: #1D1D1F;
  --apple-text-secondary: #6E6E73;
  --apple-text-tertiary: #86868B;
  --apple-text-on-dark: #F5F5F7;
  --apple-text-white: #FFFFFF;

  --apple-link-blue: #0066CC;
  --apple-link-blue-hover: #0077ED;
  --apple-accent-green: #2D8C3C;
  --apple-accent-orange: #E85D04;
  --apple-accent-red: #E30000;
}
```

### Spacing

```css
:root {
  --apple-section-gap: 100px;
  --apple-section-gap-sm: 80px;
  --apple-section-gap-lg: 120px;

  --apple-component-gap-sm: 16px;
  --apple-component-gap-md: 32px;
  --apple-component-gap-lg: 48px;

  --apple-card-padding: 32px;
  --apple-card-padding-sm: 24px;
  --apple-card-padding-lg: 40px;

  --apple-content-max-width: 980px;
  --apple-content-max-width-lg: 1200px;
}
```

### Font Weights

```css
:root {
  --apple-weight-title: 600;
  --apple-weight-title-bold: 700;
  --apple-weight-body: 400;
  --apple-weight-body-medium: 500;
}
```

### Border Radius

```css
:root {
  --apple-radius-card: 18px;
  --apple-radius-card-sm: 12px;
  --apple-radius-card-lg: 20px;
  --apple-radius-button: 980px;
  --apple-radius-badge: 8px;
}
```

### Shadows

```css
:root {
  --apple-shadow-card: 0 2px 8px rgba(0,0,0,0.04), 0 8px 24px rgba(0,0,0,0.08);
  --apple-shadow-hover: 0 4px 12px rgba(0,0,0,0.06), 0 12px 32px rgba(0,0,0,0.12);
  --apple-shadow-sm: 0 1px 4px rgba(0,0,0,0.04), 0 4px 12px rgba(0,0,0,0.06);
  --apple-shadow-modal: 0 8px 20px rgba(0,0,0,0.08), 0 20px 60px rgba(0,0,0,0.16);
}
```

### Gradients

```css
:root {
  --apple-gradient-text-purple: linear-gradient(90deg, #7B2FBE, #E040A0);
  --apple-gradient-text-blue: linear-gradient(90deg, #2997FF, #5AC8FA);
  --apple-gradient-text-warm: linear-gradient(90deg, #E8590C, #D63384);
  --apple-gradient-text-green: linear-gradient(90deg, #30D158, #00C7BE);
  --apple-gradient-bg-light: linear-gradient(180deg, #FBFBFD 0%, #F5F5F7 100%);
  --apple-gradient-bg-dark: linear-gradient(180deg, #1D1D1F 0%, #000000 100%);
}
```

Apply a text gradient:

```css
.gradient-headline {
  background: var(--apple-gradient-text-purple);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}
```

### Letter Spacing

```css
:root {
  --apple-tracking-zh-title: 0.04em;
  --apple-tracking-en-title: -0.015em;
  --apple-tracking-en-title-tight: -0.02em;
  --apple-tracking-en-title-loose: 0.01em;
  --apple-tracking-body: 0em;
}
```

Rule: `zh-CN`, `zh-TW`, `ja` → use `--apple-tracking-zh-title` for headings. `en`, `ko` → use `--apple-tracking-en-title`.

### Line Height

```css
:root {
  --apple-leading-title: 1.1;
  --apple-leading-title-tight: 1.05;
  --apple-leading-title-loose: 1.15;
  --apple-leading-body: 1.53;
  --apple-leading-body-tight: 1.5;
  --apple-leading-body-loose: 1.58;
}
```

### Responsive Breakpoints

```css
:root {
  --apple-breakpoint-sm: 734px;
  --apple-breakpoint-md: 1068px;
  --apple-breakpoint-lg: 1440px;
}
```

```css
/* Mobile-first base (< 734px) */
@media (min-width: 734px)  { /* tablet */ }
@media (min-width: 1068px) { /* desktop */ }
@media (min-width: 1440px) { /* large desktop */ }
```

---

## Typography — Multi-Language Font Stacks

### Font Stack Data

```json
{
  "fontStacks": {
    "en": {
      "display": "'SF Pro Display', 'Helvetica Neue', 'Helvetica', 'Arial', sans-serif",
      "text": "'SF Pro Text', 'Helvetica Neue', 'Helvetica', 'Arial', sans-serif",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chaos-xxl/apple-design-skill](https://github.com/chaos-xxl/apple-design-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
