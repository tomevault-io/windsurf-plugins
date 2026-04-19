---
trigger: always_on
description: This file provides guidance to Claude Code when working with the Nynaeve theme.
---

# CLAUDE.md - Nynaeve Theme

This file provides guidance to Claude Code when working with the Nynaeve theme.

## Table of Contents

1. [Development Commands](#development-commands)
2. [Block Development Philosophy](#block-development-philosophy)
3. [Creating Blocks](#creating-blocks)
4. [SVG Icons in Block Templates](#svg-icons-in-block-templates-block-bindings-pattern)
5. [Block Standards](#block-standards)
6. [Acorn Commands (Trellis VM)](#acorn-commands-trellis-vm)
7. [Architecture](#architecture)
8. [Code Standards](#code-standards)
9. [Common Tasks](#common-tasks)

---

## Efficiency
- Avoid reading entire files when only a specific section is needed
- Use `Grep` to locate relevant code before reading
- Prefer targeted reads with `offset` and `limit` parameters over full file reads

## Development Commands

```bash
cd site/web/app/themes/nynaeve
npm run dev        # Start dev server with HMR
npm run build      # Build for production
composer install && npm install
composer pint      # PHP code quality (Laravel Pint)
```

**HMR:** Use `http://imagewize.test/` (not HTTPS) — HTTPS breaks WebSocket to Vite.
**WP-CLI:** Run all `wp` / `wp acorn` commands inside Trellis VM (local DB conflicts with VM).

## Block Development Philosophy

**PREFERRED APPROACH**: Build blocks using **InnerBlocks** with native WordPress blocks whenever possible.

**Key Principles:**
- **Maximum User Control**: Users select styles, fonts, colors via block toolbar/inspector
- **Avoid Hardcoded Classes**: Never hardcode styling classes (e.g., `is-style-*`, `has-*-font-size`)
- **Native WordPress Blocks**: Use core blocks (Button, Heading, Paragraph, Image) within custom containers
- **Minimal Inspector Controls**: Only add custom controls when absolutely necessary

### When to Use Each Approach

**1. InnerBlocks (MOST PREFERRED)** — content blocks, full typography control, user-selectable styles
**2. Sage Native Blocks with Custom Controls** — dynamic JS interactivity, complex data structures
**3. ACF Composer Blocks** — repeater fields, server-side rendering, rigid brand control. See [docs/ACF-BLOCKS.md](docs/ACF-BLOCKS.md)

### InnerBlocks Best Practices

**Always use real, publishable content** in block templates — never `placeholder: 'Text goes here...'` (placeholder text only shows in the editor, not on the frontend).

**Example:**
```jsx
const TEMPLATE = [
  ['core/heading', { level: 3, content: 'Professional WordPress Development' }],
  ['core/paragraph', { content: 'Transform your website with modern development practices.' }],
  ['core/buttons', { className: 'card__buttons', layout: { type: 'flex' } }, [
    ['core/button', { text: 'Get Started' }],
    ['core/button', { text: 'Learn More' }],
  ]],
];
```

### Flex + Pseudo-element on contenteditable Elements (CRITICAL)

**Never apply `display: flex/inline-flex` with `::before`/`::after` pseudo-elements to elements WordPress uses as `contenteditable` RichText targets.**

`style.css` loads in both the frontend and the editor. Pseudo-elements on flex containers break cursor placement and make blocks impossible to click-to-edit.

**contenteditable targets (do NOT apply flex + pseudo-elements via style.css):**
- `core/paragraph` → `<p>`
- `core/heading` → `<h1>`–`<h6>`
- `core/button` → `.wp-block-button__link`
- `core/list` → `<li>`

**Fix: override in `editor.css`:**
```css
.my-block .my-styled-paragraph { display: block !important; }
.my-block .my-styled-paragraph::before { display: none !important; }
.my-block .wp-block-button__link { display: block !important; }
.my-block .wp-block-button__link::after { display: none !important; }
```

**Affected blocks (fixed):** `related-links`, `service-hero`, `trust-bar`.

---

### Button Styling (CRITICAL)

WordPress **does not reliably apply className to button links** in InnerBlocks templates. Apply className to the **parent `core/buttons` container**.

```jsx
// ❌ WRONG
['core/button', { text: 'Click Me', className: 'my-button' }]

// ✅ CORRECT
['core/buttons', { className: 'my-buttons-container', layout: { type: 'flex' } }, [
  ['core/button', { text: 'Click Me' }],
]]
```

### Block Padding (CRITICAL)

**Blocks must NOT add horizontal padding.** The WordPress layout system handles it automatically via `theme.json` root padding + `app.css` rules.

```css
/* ✅ CORRECT */
.wp-block-imagewize-my-block { padding: 5rem 0; }

/* ❌ WRONG — creates double padding */
.wp-block-imagewize-my-block { padding: 5rem 1.25rem; }
```

See [docs/CONTENT-WIDTH-AND-LAYOUT.md](docs/CONTENT-WIDTH-AND-LAYOUT.md) for full details.

### `.wp-block-paragraph` Does Not Exist on the Frontend (CRITICAL)

WordPress does **not** add the `.wp-block-paragraph` class to `<p>` elements rendered by InnerBlocks on the frontend. The class only exists in the editor. On the frontend, paragraphs render as plain `<p>` with no class (unless a custom `className` was set in the template).

**Always target `p` in block `style.css`, never `.wp-block-paragraph`:**

```css
/* ✅ CORRECT — works on both frontend and editor */
.wp-block-imagewize-my-block p {
    margin-top: 0.75rem;
    margin-bottom: 0.75rem;
}

/* ❌ WRONG — matches in editor only, invisible on frontend */
.wp-block-imagewize-my-block .wp-block-paragraph {
    margin-top: 0.75rem;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/imagewize) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
