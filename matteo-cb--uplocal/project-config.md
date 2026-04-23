---
trigger: always_on
description: Uplocal is a browser-based AI image upscaler that runs entirely client-side using WebAssembly. Zero server cost for processing. Users upload an image, it gets upscaled via Real-ESRGAN compiled to WASM inside a Web Worker, and they download the result. The core value proposition is absolute privacy (no image ever leaves the browser) and speed. Monetization via Stripe: free tier allows one upscale per day at 2x, paid unlocks 4x and 8x, batch processing, custom output formats, and unlimited usage.
---

# CLAUDE.md — Uplocal (uplocal.app)

## Project Identity

Uplocal is a browser-based AI image upscaler that runs entirely client-side using WebAssembly. Zero server cost for processing. Users upload an image, it gets upscaled via Real-ESRGAN compiled to WASM inside a Web Worker, and they download the result. The core value proposition is absolute privacy (no image ever leaves the browser) and speed. Monetization via Stripe: free tier allows one upscale per day at 2x, paid unlocks 4x and 8x, batch processing, custom output formats, and unlimited usage.

The name "Uplocal" communicates both the action (upscale) and the method (local processing). This is the primary marketing angle: your images stay on your device, always.

The site must look like it was designed by a high-end Tokyo design studio. Think Morisawa type foundry meets Vercel engineering. Every page must feel intentional, typographically rich, and quietly confident. Zero AI-generated aesthetic.

## Tech Stack

- Next.js 15 (App Router, Server Components by default)
- TypeScript strict mode everywhere
- Tailwind CSS 4
- next-intl for 10 languages
- Prisma with MongoDB (user accounts, usage tracking, subscription status)
- NextAuth.js v5 (Google, GitHub, Email magic link providers)
- Stripe Checkout + Webhooks (subscriptions and one-time credit packs)
- Real-ESRGAN NCNN compiled to WebAssembly (runs in dedicated Web Worker)
- ONNX Runtime Web as secondary/fallback upscaling engine
- next-sitemap for XML sitemap generation
- Metadata API for all SEO meta tags (no next-seo package needed)

## Absolute Rules

- NEVER use emojis anywhere in the codebase, UI, or content
- NEVER use CSS gradients of any kind (linear, radial, conic)
- NEVER use dashes in user-facing UI text (use colons, commas, or periods instead)
- NEVER use badge/pill components with rounded-full
- NEVER use Lucide React or any icon library with generic line icons
- NEVER use Inter, Roboto, Arial, or system font stacks as design fonts
- NEVER use purple as a primary or accent color
- NEVER use drop shadows on cards as the primary visual treatment
- NEVER put placeholder "Lorem ipsum" in shipped pages
- ALL images must have explicit width, height, and alt text
- ALL pages must have unique, keyword-rich titles and descriptions per locale
- ALL interactive elements must have visible focus states
- ALL forms must have proper labels and error handling

## Design System

### Philosophy

"Japanese editorial minimalism." Clean, asymmetric layouts. Bold typographic hierarchy. Sharp geometry. Intentional whitespace used as a design element. Restrained color palette with one warm accent. The design should feel like opening a well-made architecture magazine, not a SaaS landing page.

### Typography

Display font: "Instrument Serif" from Google Fonts. Elegant, editorial, unexpected for a tech product. Used for all headings h1 through h3.

Body font: "Satoshi" from Fontshare CDN. Geometric sans-serif, modern, excellent readability at all sizes. Used for body text, UI labels, navigation, buttons.

Monospace: "JetBrains Mono" from Google Fonts. Used sparingly for technical specs, file sizes, resolution numbers.

Font loading: Use `next/font/google` for Instrument Serif and JetBrains Mono. Use `next/font/local` for Satoshi (self-hosted woff2). Display swap for headings, optional for body to prevent layout shift.

### Color Palette

```css
:root {
  --color-ink: #0A0A0A;
  --color-paper: #F6F4F0;
  --color-accent: #C2410C;
  --color-accent-hover: #9A3412;
  --color-accent-light: #FFF7ED;
  --color-muted: #78716C;
  --color-border: #D6D3D1;
  --color-surface: #FAFAF9;
  --color-error: #B91C1C;
  --color-success: #15803D;
}

[data-theme="dark"] {
  --color-ink: #FAFAF9;
  --color-paper: #0C0A09;
  --color-accent: #EA580C;
  --color-accent-hover: #F97316;
  --color-accent-light: #1C1917;
  --color-muted: #A8A29E;
  --color-border: #292524;
  --color-surface: #1C1917;
  --color-error: #EF4444;
  --color-success: #22C55E;
}
```

### Spacing Scale

Use Tailwind default spacing. Sections use py-24 or py-32 for generous vertical rhythm. Max content width is max-w-6xl. The upscaler tool itself uses max-w-4xl centered.

### Component Patterns

Buttons: Rectangular with sharp corners (rounded-none or rounded-sm max). Primary button uses bg-accent with text-white. Secondary uses border with border-ink. Hover states use background color shift, never opacity change. All buttons have min-height of 48px for touch targets.

Cards: Use border with border-color-border. Background is color-surface. No shadow. On hover, border shifts to color-ink. Transition duration 200ms.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Matteo-CB) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
