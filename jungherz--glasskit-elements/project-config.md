---
trigger: always_on
description: GlassKit Elements is a vanilla-JS Web Components library (v1.6.0) wrapping GlassKit CSS v1.6.0. It provides 29 custom elements with the `glk-` prefix, Dark/Light mode with automatic theme sync, Shadow DOM encapsulation, and form-associated custom elements. Use this reference whenever generating HTML that uses `<glk-*>` tags to ensure correct attributes, slots, events, and composition.
---


# GlassKit Elements – AI Component Reference

> **Purpose:** AI-optimized reference for generating correct `<glk-*>` markup.
> Companion to the class-based `SKILL.md` in [`@jungherz-de/glasskit`](https://github.com/JUNGHERZ/GlassKit) — the element wrappers delegate visuals and tokens to GlassKit CSS, so both references are best used together.

---

## 1. Setup & Boilerplate

### Installation

```bash
npm install @jungherz-de/glasskit-elements @jungherz-de/glasskit
```

Peer dependency `@jungherz-de/glasskit >=1.6.0` is required. The v1.6.0 floating Tab-Bar variant + Accessory capsule (`<glk-tab-dock>`, `<glk-tab-accessory>`, `<glk-tab-bar floating>`) requires CSS v1.6.0.

### Import (ES modules)

```js
// Full bundle — registers all 29 elements
import '@jungherz-de/glasskit-elements';

// Named imports (for direct references to constructor classes)
import { GlkButton, GlkModal } from '@jungherz-de/glasskit-elements';

// Tree-shaken per-component import
import { GlkButton } from '@jungherz-de/glasskit-elements/components/glk-button.js';
```

### CDN

```html
<script src="https://cdn.jsdelivr.net/npm/@jungherz-de/glasskit-elements/dist/glasskit-elements.min.js"></script>
```

Note: the elements bundle the GlassKit CSS `CSSStyleSheet` via `adoptedStyleSheets`. You do **not** need to load `glasskit.css` separately — it is already inside every component's shadow root.

### Minimal template

```html
<!DOCTYPE html>
<html lang="en" data-theme="dark">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <script src="https://cdn.jsdelivr.net/npm/@jungherz-de/glasskit-elements/dist/glasskit-elements.min.js"></script>
</head>
<body>
  <div class="glass-bg">
    <glk-title>Hello GlassKit</glk-title>
    <glk-card>
      <p>Welcome!</p>
    </glk-card>
  </div>
</body>
</html>
```

The outer `.glass-bg` class (from GlassKit CSS) still provides the aurora background. Inside the shadow roots, each element carries its own glass styling.

### Theme

```html
<html data-theme="dark">  <!-- or "light" -->
```

```js
function toggleTheme() {
  const html = document.documentElement;
  const current = html.getAttribute('data-theme') || 'dark';
  html.setAttribute('data-theme', current === 'dark' ? 'light' : 'dark');
}
```

A single module-level `MutationObserver` watches `data-theme` on `<html>` and syncs every live GlassKit Element instance — you never set `data-theme` on individual elements.

---

## 2. Core Concepts

| Concept | Summary |
|---|---|
| Tag prefix | `glk-*` (analogous to the `glass-*` CSS prefix) |
| Rendering | Shadow DOM (`mode: 'open'`) with `adoptedStyleSheets` |
| Stylesheet sharing | GlassKit's `glassSheet` is the same `CSSStyleSheet` object in every element — no CSS duplication |
| Theme sync | Global MutationObserver on `<html data-theme>` |
| Events | Custom `glk-*` events, all `bubbles: true, composed: true` |
| Form participation | `GlkFormElement` uses `ElementInternals` (`static formAssociated = true`) |
| API style | Declarative HTML attributes + reflected JS properties |

Custom properties (`--gl-*`) defined on `:root` or `<html>` pass through shadow boundaries automatically, so custom theming works with a single global stylesheet.

---

## 3. Element Catalog (29 elements)

### 3.1 `<glk-nav>`

Horizontal navigation bar — flex container. Typically holds `<glk-pill>` buttons.

```html
<glk-nav>
  <glk-pill label="Back" onclick="history.back()">
    <svg viewBox="0 0 24 24"><polyline points="15 18 9 12 15 6"/></svg>
  </glk-pill>
  <glk-pill label="Settings">
    <svg viewBox="0 0 24 24">...</svg>
  </glk-pill>
</glk-nav>
```

No attributes. Default slot for children.

---

### 3.2 `<glk-pill>`

Circular icon button (46×46 px).

```html
<glk-pill label="Back">
  <svg viewBox="0 0 24 24"><polyline points="15 18 9 12 15 6"/></svg>
</glk-pill>
```

| Attribute | Type | Description |
|---|---|---|
| `label` | String | `aria-label` for accessibility |
| `disabled` | Boolean | Disabled state |

Default slot: icon SVG (24×24).

Events: `glk-click` when clicked.

---

### 3.3 `<glk-tab-bar>` + `<glk-tab-item>`

Fixed bottom navigation. Requires `.glass-bg--has-tab-bar` on the outer background to reserve padding.

```html
<div class="glass-bg glass-bg--has-tab-bar">
  <!-- page content -->
  <glk-tab-bar>
    <glk-tab-item label="Home" active>
      <svg viewBox="0 0 24 24"><path d="M3 9l9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"/></svg>
    </glk-tab-item>
    <glk-tab-item label="Search">
      <svg viewBox="0 0 24 24"><circle cx="11" cy="11" r="8"/></svg>
    </glk-tab-item>
    <glk-tab-item label="Profile" badge="3">
      <svg viewBox="0 0 24 24"><path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/></svg>
    </glk-tab-item>
  </glk-tab-bar>
</div>
```

**`<glk-tab-bar>`** — container.

| Attribute | Type | Description |
|---|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JUNGHERZ/GlassKit-Elements](https://github.com/JUNGHERZ/GlassKit-Elements) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
