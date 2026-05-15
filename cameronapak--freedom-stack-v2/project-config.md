---
trigger: always_on
description: Basecoat UI CSS LLMS.txt
---


# Basecoat - llms.txt

Basecoat is a components library built with Tailwind CSS that works with any web stack. It brings the magic of shadcn/ui to any traditional web stack with no React required.

## Links

- Main documentation: https://basecoatui.com/
- Introduction: https://basecoatui.com/introduction/
- Installation guide: https://basecoatui.com/installation/
- GitHub repository: https://github.com/hunvreus/basecoat
- Kitchen Sink (all components): https://basecoatui.com/kitchen-sink/
- NPM package: https://www.npmjs.com/package/basecoat-css

## Overview

Basecoat provides modern, accessible components with the simplicity of plain HTML and Tailwind. It is:

- Lightweight: No runtime JS, just CSS and minimal vanilla JavaScript
- Framework-agnostic: Works with any backend or frontend stack
- Accessible: Components follow accessibility best practices
- Dark mode ready: Respects your Tailwind config
- Themable: Fully compatible with shadcn/ui themes
- Free and open source: MIT licensed

## Installation

### CDN Installation

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/basecoat-css@0.2.8/dist/basecoat.cdn.min.css" />
<script src="https://cdn.jsdelivr.net/npm/basecoat-css@0.2.8/dist/js/all.min.js" defer></script>
```

### NPM Installation

```bash
npm install basecoat-css
```

```css
@import "tailwindcss";
@import "basecoat-css";
```

### Requirements

- Tailwind CSS (required dependency)
- No other dependencies for CSS-only components
- JavaScript files required only for interactive components (dropdown-menu, popover, select, sidebar, tabs, toast)

## Usage Rules

- Add component classes directly to HTML elements (e.g., `btn`, `card`, `input`)
- Combine with Tailwind utility classes for customization
- Use semantic HTML elements as the foundation
- JavaScript components require specific HTML structure and attributes
- Components are designed to work without build tools

## Configuration and Theming

### Theme Compatibility

Basecoat is fully compatible with shadcn/ui themes. Import any shadcn/ui theme:

```css global.css
@import "tailwindcss";
@import "basecoat-css";
@import "./theme.css"; /* shadcn/ui theme variables */
```

### Customization

- Override styles using Tailwind utility classes
- Use CSS variables for deeper customization
- Copy basecoat.css file for extensive modifications
- Extend or override styles in custom CSS files

## Components Reference

### Accordion

**Classes:** `accordion`
**Structure:** Uses `<details>` and `<summary>` elements with `accordion` wrapper class
**JavaScript:** Optional (included for single-panel behavior)
**Usage:**

```html
<section class="accordion">
  <details class="group border-b">
    <summary>
      <h2>Question Title</h2>
    </summary>
    <section class="pb-4">Answer content</section>
  </details>
</section>
```

### Alert

**Classes:** `alert`, `alert-destructive`
**Structure:** Container with optional icon, heading, and description
**Usage:**

```html
<div class="alert">
  <svg>...</svg>
  <h2>Alert Title</h2>
  <section>Alert description</section>
</div>
```

### Alert Dialog

**Classes:** `dialog`
**Structure:** Uses native `<dialog>` element
**JavaScript:** Uses native browser APIs
**Usage:**

```html
<dialog id="alert-dialog" class="dialog">
  <article>
    <header>
      <h2>Title</h2>
      <p>Description</p>
    </header>
    <footer>
      <button class="btn-outline">Cancel</button>
      <button class="btn-primary">Continue</button>
    </footer>
  </article>
</dialog>
```

### Avatar

**Implementation:** No dedicated class - use Tailwind utilities
**Usage:**

```html
<img class="size-8 shrink-0 rounded-full object-cover" alt="User" src="avatar.png" />
```

### Badge

**Classes:** `badge`
**Variants:** Default styling, combine with Tailwind utilities for colors and sizes
**Usage:**

```html
<span class="badge">New</span>
```

### Breadcrumb

**Implementation:** No dedicated class - use Tailwind utilities with optional dropdown-menu
**Usage:**

```html
<ol class="text-muted-foreground flex flex-wrap items-center gap-1.5 text-sm">
  <li><a href="#" class="hover:text-foreground">Home</a></li>
  <li><svg>chevron</svg></li>
  <li><span class="text-foreground">Current Page</span></li>
</ol>
```

### Button

**Classes:** `btn`, `btn-primary`, `btn-secondary`, `btn-destructive`, `btn-outline`, `btn-ghost`, `btn-link`, `btn-icon`
**Sizes:** `btn-sm`, `btn-lg` (can combine: `btn-lg-destructive`, `btn-sm-icon-outline`)
**Usage:**

```html
<button class="btn-primary">Primary Button</button>
<button class="btn-sm-outline">Small Outline Button</button>
<button class="btn-icon"><svg>icon</svg></button>
```

### Card

**Classes:** `card`
**Structure:** Container with optional header, content, and footer
**Usage:**

```html
<div class="card">
  <header>
    <h3>Card Title</h3>
    <p>Card description</p>
  </header>
  <section>Card content</section>
  <footer>Card footer</footer>
</div>
```

### Checkbox

**Classes:** Standard HTML checkbox with Basecoat styling
**Usage:**

```html
<input type="checkbox" id="checkbox" /> <label for="checkbox">Checkbox label</label>
```

### Dialog

**Classes:** `dialog`
**JavaScript:** Required for functionality
**Structure:** Native `<dialog>` element with specific markup
**Usage:**

```html
<dialog class="dialog" id="dialog">

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cameronapak/freedom-stack-v2](https://github.com/cameronapak/freedom-stack-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
