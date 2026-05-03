---
trigger: always_on
description: This is the marketing/documentation website for Default Tamer, a macOS browser routing app. Built with **Astro**, **Tailwind CSS v3**, and **native HTML interactions** powered by a lightweight script.
---

# Default Tamer Website — Copilot Instructions

## Project Overview

This is the marketing/documentation website for Default Tamer, a macOS browser routing app. Built with **Astro**, **Tailwind CSS v3**, and **native HTML interactions** powered by a lightweight script.

---

## Mandatory: Use Native HTML Interactions

**When creating or updating interactive UI, use native HTML patterns first (`<details>`, `<dialog>`, semantic buttons/links) and wire behavior through the shared script at `public/scripts/custom-interactions.js`.**

Do not re-introduce `@tailwindplus/elements` or `el-*` tags unless explicitly requested by the maintainer.

### Interaction Rules

| Need | Preferred Pattern | Avoid |
|------|-------------------|-------|
| Show/hide content (FAQ, accordion, collapsible) | Native `<details>` + `<summary>` | Custom div toggles with ad-hoc state |
| Modal/dialog | Native `<dialog>` with `data-dialog-trigger` / `data-dialog-close` hooks | `display:none` modal implementations |
| Copy to clipboard | `data-copy` button targeting an element ID | Per-page custom clipboard scripts |
| Keyboard-accessible menus/tabs | Reuse handlers in `custom-interactions.js` | Duplicated one-off keyboard logic |

### Current Interaction Architecture

- Shared script: `DefaultTamerWeb/public/scripts/custom-interactions.js`
- Global behavior styles: `DefaultTamerWeb/src/styles/global.css` (native `dialog` and `details` styling)
- Base layout loader: `DefaultTamerWeb/src/layouts/BaseLayout.astro` loads the shared interaction script

### Native FAQ Pattern

```html
<details class="group bg-white border-2 border-gray-light rounded-xl overflow-hidden hover:border-primary transition-colors duration-300">
  <summary class="flex items-center justify-between p-6 cursor-pointer font-semibold text-lg list-none">
    <span class="group-open:text-primary transition-colors duration-300">Question?</span>
    <span class="text-primary text-2xl group-open:rotate-45 transition-transform duration-300">+</span>
  </summary>
  <div class="px-6 pb-6 text-gray leading-relaxed">
    Answer content.
  </div>
</details>
```

### Native Dialog Pattern

```html
<button data-dialog-trigger="dialog-id">Open</button>

<dialog id="dialog-id">
  <div class="relative z-10">
    <button data-dialog-close>✕</button>
    <!-- Content -->
  </div>
  <div class="fixed inset-0 bg-black/50"></div>
</dialog>
```

### Clipboard Pattern (Code Blocks)

```html
<pre id="snippet">npm install default-tamer</pre>

<button data-copy="snippet">Copy</button>
```

---

## Styling Rules

### ALWAYS Use Tailwind Utilities

- **NEVER** write scoped `<style>` blocks in `.astro` files
- **NEVER** create new CSS classes when Tailwind utilities exist
- **ALWAYS** use Tailwind utility classes directly in HTML
- **EXCEPTION:** Shared component classes in `global.css` (`.card-animated`, `.callout-*`, `.docs-wrapper`, `.docs-page`, `.docs-content`, `.prose`, `.hero-section`, and doc-specific component classes)

### Brand Colors (from tailwind.config.mjs)

| Token | Value | Usage |
|-------|-------|-------|
| `primary` | `#f97316` | Links, CTAs, active states, accents |
| `primary-dark` | `#ea580c` | Hover states for primary |
| `secondary` | `#10b981` | Gradient accent (rarely alone) |
| `dark` | `#1e293b` | Text, dark backgrounds |
| `dark-light` | `#334155` | Hover for dark elements |
| `gray` | `#475569` | Body text, secondary text |
| `gray-light` | `#e2e8f0` | Borders, dividers |
| `gray-lighter` | `#f8fafc` | Background surfaces |

### Typography

- **Headings:** `font-heading` (Outfit)
- **Body:** `font-sans` (Plus Jakarta Sans) — applied via base layer
- **Never** add font-family inline or import other fonts

### Component Patterns

| Pattern | Classes |
|---------|---------|
| Card | `card-animated p-6 bg-white border-2 border-gray-light rounded-xl hover:border-primary hover:-translate-y-2 transition-all duration-300 hover:shadow-xl` |
| Hero section | `hero-section` (defined in global.css) |
| Info callout | `callout-info` |
| Warning callout | `callout-warning` |
| Success callout | `callout-success` |
| Docs wrapper (prose pages) | `docs-wrapper prose` |
| Docs page (sidebar layout) | `docs-page` + `docs-content` (used in troubleshooting, advanced-usage) |
| Primary button | `rounded-lg bg-primary px-6 py-3 text-sm font-semibold text-dark hover:bg-primary-dark transition-all duration-200` |
| Ghost link | `text-primary font-semibold hover:text-primary-dark transition-colors duration-200` |

---

## Page Structure

### Layout Hierarchy

```
BaseLayout.astro
├── SEO.astro (meta tags)
├── Google Fonts (Outfit + Plus Jakarta Sans)
├── custom-interactions.js (async)
├── Header.astro (sticky nav with native dialog/menu patterns)
├── <main><slot /></main>
├── Footer.astro
├── SearchPalette.astro (⌘K search using native dialog)
└── Back to top button (vanilla JS scroll listener)
```

### Page Types

1. **Marketing pages** (`index.astro`, `features.astro`, `download.astro`) — Full-width sections with `hero-section`, feature grids, CTAs
2. **Docs hub** (`docs/index.astro`) — Cards grid + quick links + FAQ with native `<details>`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0xdps/default-tamer](https://github.com/0xdps/default-tamer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
