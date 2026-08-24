---
trigger: always_on
description: vael is a web-based design system generator built for freelance designers and SaaS product teams. It takes brand parameters as input and extrapolates a complete, production-ready design token system using the Anthropic Claude API.
---

# vael — CLAUDE.md

## What is vael?

vael is a web-based design system generator built for freelance designers and SaaS product teams. It takes brand parameters as input and extrapolates a complete, production-ready design token system using the Anthropic Claude API.

The long-term goal is to make vael a shareable tool for the broader design community — a BYOK (bring your own key) web app.

---

## Critical rules — read before every task

### Build verification (mandatory)
After every task that modifies existing files, run:
```
npm run build
```
If the build fails, show the exact error and fix it before marking the task complete. Do not report a task as done if the build has errors. A passing build prevents white screen crashes in the browser.

### Auto-compact threshold
Context window auto-compacts at 30% remaining. This is intentional — do not fight it.

### Scope discipline
Only modify files explicitly mentioned in the prompt. Do not touch unrelated files even if you notice issues. If you see a problem outside the scope, note it but do not fix it without being asked.

### Commit reminders
Always remind Neil to commit to git before starting a task and after completing it.

---

## Core interaction model

**Two-panel layout:**
- Left panel: live component preview canvas
- Right panel: parameter controls + Generate button

**Two modes:**

1. **Live preview (no API call)** — the canvas updates in real time as the user adjusts parameters. Color pickers, radius chips, density toggles, dark mode switch all update the canvas instantly. This is a direct translation of raw inputs into rendered components — no Claude involvement.

2. **Generate (API call)** — clicking Generate sends the parameters to the Claude API. Claude extrapolates the full token system: complete color scales (50–950), semantic colors, spacing ramp, shadow scale, motion tokens, and component-level tokens. The canvas updates to reflect the generated system. Export becomes available after generation.

**Post-generate tweaks** apply locally on top of the generated system without a new API call. The user must click Generate again to fully recalculate.

---

## Parameter set

| Section | Parameters |
|---|---|
| Brand | name, primary color, accent color, tone |
| Typography | display font, body font |
| Geometry | border radius scale, spacing density |
| Motion | motion style, easing (enter/exit/hover), speed multiplier, interaction preset |
| Interactions | preset, trigger, target |
| Output | dark mode toggle, export format |

**Tone options:** professional, playful, editorial, minimal, bold, enterprise

**Radius options:** sharp, subtle, moderate, rounded, pill

**Density options:** compact, comfortable, spacious

**Motion options:** none, subtle, expressive

**Interaction presets:** border trace, fill sweep, shimmer, pulse ring, magnetic

**Export formats:** tailwind.config.js, CSS variables, Tokens Studio JSON

---

## Token architecture

### Pre-generate (live preview)
Before Generate is clicked, the canvas is driven directly by raw param values:
- Primary color → used as-is for buttons, focus rings, accents
- Accent color → used as-is for highlights
- Dark mode → toggles between dark/light surface values
- Radius scale → maps to a fixed lookup table of pixel values
- Density → maps to a fixed spacing multiplier

### Post-generate (full system)
After Generate, the canvas is driven by the Claude-generated token object:

```js
{
  colors: {
    primary:  { 50, 100, 200, 300, 400, 500, 600, 700, 800, 900 },
    accent:   { 50, 100, 200, 300, 400, 500, 600, 700, 800, 900 },
    neutral:  { 0, 50, 100, 200, 300, 400, 500, 600, 700, 800, 900, 950, 1000 },
    semantic: { success, warning, error, info }
  },
  typography: {
    displayFont, bodyFont,
    scale:         { xs, sm, base, lg, xl, 2xl, 3xl, 4xl, 5xl },
    weight:        { light, regular, medium, semibold, bold },
    lineHeight:    { tight, snug, normal, relaxed },
    letterSpacing: { tight, normal, wide, wider }
  },
  spacing: { px, 1, 2, 3, 4, 5, 6, 8, 10, 12, 16, 20, 24 },
  radius:  { none, xs, sm, md, lg, xl, 2xl, full },
  shadow:  { xs, sm, md, lg, xl },
  motion: {
    duration: { instant, fast, normal, slow, slower },
    easing:   { default, enter, exit, spring }
  },
  components: {
    button: { paddingX, paddingY, fontSize, fontWeight, radius, primaryBg, primaryText, primaryHover },
    input:  { paddingX, paddingY, fontSize, radius, borderColor, bg, focusBorder },
    card:   { padding, radius, bg, borderColor, shadow },
    badge:  { paddingX, paddingY, fontSize, radius, primaryBg, primaryText }
  }
}
```

---

## Token naming convention

All generated tokens follow this structured naming schema:

### Primitives — `{category}-{scale}`
- `color-primary-500`
- `font-size-lg`
- `spacing-4`
- `radius-md`

### Semantics — `{role}-{variant}-{state}`
- `surface-default`
- `text-primary`
- `border-focus`
- `feedback-success`

### Component tokens — `{component}-{element}-{property}-{state}`
- `button-primary-bg-default`
- `button-primary-bg-hover`
- `input-border-focus`
- `card-surface-bg-default`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bigneilenergy/vael](https://github.com/bigneilenergy/vael) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
