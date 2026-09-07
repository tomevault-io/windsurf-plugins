---
trigger: always_on
description: This file describes the structure, conventions, and rules for building new tools in this collection. Read it before creating or modifying any project.
---

# AGENTS.md — Development Guide

This file describes the structure, conventions, and rules for building new tools in this collection. Read it before creating or modifying any project.

---

## What This Repo Is

A collection of standalone, self-contained web tools. Each tool lives in its own directory and is deployed as a static page. The root `index.html` is auto-generated from metadata found in each tool's `index.html`.

---

## Rules for New Projects

### 1. No React, No Tailwind

Do not use any UI frameworks or utility CSS libraries. Write plain HTML, CSS, and JavaScript.

- **CSS**: Use modern CSS — nesting, custom properties (variables), `@layer`, `color-mix()`, `:has()`, container queries, etc.
- **JS**: Vanilla JavaScript only. No bundlers, no npm, no build step (unless the project genuinely requires one, like `movies/`).
- **Fonts**: Use system font stacks. No web fonts loaded from external CDNs.

### 2. Mobile-First Design with Dark and Light Mode

- Write styles for mobile first, then add `@media (min-width: ...)` overrides for larger screens.
- Every project must support both dark and light colour schemes.
- **Follow the system setting.** The theme comes from the OS/browser light–dark control via `prefers-color-scheme` — nothing else.
- **Do not add a theme toggle.** A manual light/dark switch is not required and should not be built unless the author explicitly asks for one. Without an explicit request there is no toggle button, no `theme` value in `localStorage`, and no `?theme=` URL parameter — the system control is the only input.
- If a toggle *is* explicitly requested, layer it on top with a `[data-theme]` attribute on `<html>`, keeping `prefers-color-scheme` as the default when the attribute is absent.
- Define all colours as CSS custom properties in `:root` so themes can be swapped cleanly.
- Set `color-scheme: light dark` on `:root` so native controls, form fields, and scrollbars follow along too.

**Minimal theme pattern:**

```css
:root {
  color-scheme: light dark;

  --bg: #fafafa;
  --bg-secondary: #f4f4f5;
  --border: #d4d4d8;
  --text: #18181b;
  --text-secondary: #52525b;
  --accent: #2563eb;
}

@media (prefers-color-scheme: dark) {
  :root {
    --bg: #09090b;
    --bg-secondary: #18181b;
    --border: #3f3f46;
    --text: #fafafa;
    --text-secondary: #a1a1aa;
    --accent: #60a5fa;
  }
}
```

Every colour that differs between the two schemes belongs in this block as a custom property. Avoid one-off `body.light .thing { color: ... }` overrides scattered through the stylesheet — add a variable instead and give it a value in each scheme.

### 3. Each Project Lives in Its Own Directory

Create a subdirectory at the repo root named after the tool, using kebab-case:

```
repo-root/
└── my-new-tool/
    ├── index.html      ← required
    ├── style.css       ← preferred default
    └── script.js       ← preferred when JS is used
```

Keep the tool self-contained. Avoid referencing files outside the tool's directory, apart from the shared root directories listed under "Shared Code" below.
Prefer smaller, focused files over one large file. Default to splitting HTML, CSS, and JS into separate files.
Inline `<style>`/`<script>` blocks should be treated as exceptions for very small throwaway prototypes only.

### 4. Required Meta Tags in `index.html`

The index generator (`scripts/generate_index.py`) reads each tool's `index.html` to build the main listing page. Two meta tags are **required**:

```html
<meta name="description" content="One or two sentence description of what this tool does.">
<meta name="category" content="CategoryName">
```

Without these the tool will appear as "Uncategorized" and may have no description on the index page.

**Valid categories** (match existing ones or add a new one consistently):

| Category          | Good fit when…                                                                                                                                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `Developer Tools` | The primary user is a developer; the tool helps with code, data formats, APIs, dependencies, diffs, or hardware specs (e.g. JSON validator, package browser, merge tool, ESP32 comparison) |
| `Calculators`     | The tool takes numeric inputs and produces a computed result — rates, values, durations, or unit conversions (e.g. capacitor decoder, tax calculator, time adder)                          |
| `Game`            | The tool is an interactive game with scoring, winning, or challenge mechanics (e.g. quiz, puzzle, countdown numbers)                                                                       |
| `Home Assistant`  | The tool queries, debugs, or integrates with a Home Assistant instance or its config format                                                                                                |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [remy/tools](https://github.com/remy/tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
