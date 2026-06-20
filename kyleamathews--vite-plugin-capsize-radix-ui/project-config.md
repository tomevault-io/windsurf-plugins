---
trigger: always_on
description: Set up pixel-perfect typography for Radix UI using vite-plugin-capsize-radix. Helps configure fonts, choose pairings, and integrate with your project.
---


# Radix Typography with Capsize

This skill helps you set up bulletproof typography for Radix UI applications using `vite-plugin-capsize-radix`. It generates CSS that trims the invisible whitespace above and below text (caused by font metrics), enabling pixel-perfect alignment.

## What This Plugin Does

Traditional CSS font rendering includes invisible space above and below text based on the font's ascender/descender metrics. This makes precise layouts difficult. Capsize calculates these metrics and applies CSS pseudo-elements to trim the excess, so your text sits exactly where you specify.

This plugin:
- Generates CSS that overrides Radix UI's typography variables
- Creates font stacks with metric-aware fallbacks
- Supports separate fonts for body text, headings, and code
- Handles responsive sizing (mobile vs desktop)

## Important: Spacing with Capsize

Because Capsize trims the invisible whitespace above and below text, **adjacent text elements will sit directly against each other** without any natural spacing. This is by design - it gives you precise control over layout.

You must explicitly add spacing between text elements using:
- **Flex with gap** (recommended): `<Flex direction="column" gap="2">`
- **margin/padding**: Apply to individual elements
- **Radix spacing props**: `mt`, `mb`, `my`, etc.

```tsx
// Without gap - heading and text will touch
<Flex direction="column">
  <Heading>Recipes</Heading>
  <Text>Define your chocolate recipes...</Text>
</Flex>

// With gap - natural spacing between elements
<Flex direction="column" gap="2">
  <Heading>Recipes</Heading>
  <Text>Define your chocolate recipes...</Text>
</Flex>
```

This explicit spacing model is more predictable than relying on font metrics and gives you pixel-perfect control over your layouts.

## Preserving Descenders

Capsize trims text to the cap-height (top) and alphabetic baseline (bottom). This can clip descenders (the tails of letters like "g", "y", "p") on large headings with tight line-heights.

Add the `preserve-descenders` class to keep descenders visible:

```tsx
<Heading size="9" className="preserve-descenders">
  Signal Sky
</Heading>
```

This disables the bottom trim while keeping the top tight. Use this on large display headings where you notice descender clipping.

## Installation

```bash
pnpm add vite-plugin-capsize-radix @capsizecss/metrics
```

## Quick Start

### 1. Configure the Plugin

In `vite.config.ts`:

```typescript
import { defineConfig } from "vite"
import react from "@vitejs/plugin-react"
import { capsizeRadixPlugin } from "vite-plugin-capsize-radix"

// Import metrics for your chosen fonts
import inter from "@capsizecss/metrics/inter"
import arial from "@capsizecss/metrics/arial"

export default defineConfig({
  plugins: [
    react(),
    capsizeRadixPlugin({
      outputPath: `./public/typography.css`,
      defaultFontStack: [inter, arial],
    }),
  ],
})
```

### 2. Install Fonts via Fontsource

For open source fonts, use [@fontsource](https://fontsource.org/):

```bash
pnpm add @fontsource/inter
```

### 3. Import CSS in Correct Order

In your entry file (e.g., `main.tsx`), the import order matters:

```typescript
// 1. Radix base styles FIRST
import "@radix-ui/themes/styles.css"

// 2. Your font CSS from Fontsource
import "@fontsource/inter/latin.css"

// 3. Generated typography CSS LAST (overrides Radix variables)
import "/typography.css"

// 4. Your app styles
import "./App.css"
```

**Critical**: The generated typography CSS must come AFTER `@radix-ui/themes/styles.css` to properly override the CSS variables.

## Using Fontsource Fonts

Fontsource provides npm packages for Google Fonts and other open source fonts. Each package includes the font files and CSS.

### Installation Pattern

```bash
# Install the font package
pnpm add @fontsource/[font-name]

# Install the metrics (usually same name, check @capsizecss/metrics)
# Metrics are included in @capsizecss/metrics
```

### Import Patterns

```typescript
// Full font (all weights, styles, subsets)
import "@fontsource/inter"

// Specific subset (recommended for performance)
import "@fontsource/inter/latin.css"

// Specific weight
import "@fontsource/inter/400.css"
import "@fontsource/inter/700.css"

// Variable font (if available)
import "@fontsource-variable/inter"
```

### Common Fontsource Packages

| Font | Package | Metrics Import |
|------|---------|----------------|
| Albert Sans | `@fontsource/albert-sans` | `@capsizecss/metrics/albertSans` |
| Alegreya | `@fontsource/alegreya` | `@capsizecss/metrics/alegreya` |
| Alegreya Sans | `@fontsource/alegreya-sans` | `@capsizecss/metrics/alegreyaSans` |
| Archivo | `@fontsource/archivo` | `@capsizecss/metrics/archivo` |
| Arvo | `@fontsource/arvo` | `@capsizecss/metrics/arvo` |
| Atkinson Hyperlegible | `@fontsource/atkinson-hyperlegible` | `@capsizecss/metrics/atkinsonHyperlegible` |
| Be Vietnam Pro | `@fontsource/be-vietnam-pro` | `@capsizecss/metrics/beVietnamPro` |
| Bodoni Moda | `@fontsource/bodoni-moda` | `@capsizecss/metrics/bodoniModa` |
| Bricolage Grotesque | `@fontsource/bricolage-grotesque` | `@capsizecss/metrics/bricolageGrotesque` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KyleAMathews/vite-plugin-capsize-radix-ui](https://github.com/KyleAMathews/vite-plugin-capsize-radix-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
