---
trigger: always_on
description: This directory contains the web layer source bundled with MarkdownView
---

# webassets — Agent Guide

This directory contains the web layer source bundled with MarkdownView
(an iOS WKWebView library), along with its build and test environment.

---

## Directory Structure

```
webassets/
├── build.mjs                  # esbuild build script
├── package.json               # Dependencies & npm scripts
├── playwright.config.js       # Playwright test configuration
├── src/
│   ├── js/
│   │   ├── index.js           # Full JS entry point (113 languages + renderer)
│   │   ├── index-core.js      # Core JS entry point (15 common languages + renderer)
│   │   ├── index-extended.js  # Extended-languages-only bundle (97 additional languages)
│   │   └── render.js          # Shared renderer logic (markdown-it, height notification)
│   └── css/
│       ├── bootstrap.css      # Bootstrap v3.3.7 (only .table / .container rules)
│       ├── gist.css           # highlight.js theme (gist)
│       ├── github.css         # GitHub Markdown styles
│       └── index.css          # Custom CSS (CSS variables & dark mode support)
└── tests/
    └── render.spec.js         # Playwright functional tests (18 cases)
```

Build artifacts are output **outside of `webassets/`**.

```
../Sources/MarkdownView/Resources/
├── main.js           # Full bundle (~715 KB) — all 113 languages + renderer (for tests)
├── main-core.js      # Core bundle (~175 KB) — 15 common languages + renderer (inlined into HTML)
├── main-core.css     # CSS bundle (inlined into HTML)
├── main-extended.js  # Extended-languages bundle (~540 KB) — 97 additional languages only
├── styled.html       # Self-contained HTML template with inlined core JS/CSS (styled)
└── non_styled.html   # Self-contained HTML template with inlined core JS (non-styled)
```

---

## npm Scripts

| Command | Description |
|---------|-------------|
| `npm run build` | Bundle & minify with esbuild, output to `Sources/MarkdownView/Resources/` |
| `npm test` | Run headless Chromium tests with Playwright |

Installing dependencies is only required on first setup.

```sh
cd webassets
npm install
npm run build
npm test
```

---

## Build Configuration (build.mjs)

- **Bundler**: esbuild (replaces webpack + Babel)
- **Target**: `safari13` (supports WKWebView; app deployment target iOS 16+)
- **Output**: IIFE format, minified
- **License comments**: `legalComments: 'none'` (no LICENSE.txt generated)

Build time is typically under 100 ms.

---

## JS Entry Points

### Dependencies

| Library | Version | Purpose |
|---------|---------|---------|
| `highlight.js` | ^11.11.1 | Syntax highlighting |
| `markdown-it` | ^14.1.0 | Markdown parsing & rendering |
| `markdown-it-emoji` | ^3.0.0 | Emoji shortcode conversion |

### Bundle Architecture

The JS is split into three bundles for performance optimization:

| Entry Point | Output | Languages | Includes Renderer |
|-------------|--------|-----------|-------------------|
| `index-core.js` | `main-core.js` | 15 common | Yes (via `render.js`) |
| `index.js` | `main.js` | 113 all | Yes (via `render.js`) |
| `index-extended.js` | `main-extended.js` | 97 additional | No (registers on `window._hljs`) |

- **`main-core.js`** is inlined into HTML templates at build time for fast initial load
- **`main-extended.js`** is injected via `evaluateJavaScript` after first render to add remaining languages
- **`main.js`** is the full bundle used by Playwright tests

### highlight.js Language Set

**113 languages are individually imported** into `highlight.js/lib/core`
(changed from bundling all 192 languages to reduce bundle size).

To add or remove languages, edit both the import statement and the
`hljs.registerLanguage()` call in the relevant `index*.js` file as a pair.

**Note**: `import markdownLang from "highlight.js/lib/languages/markdown"` uses
the alias `markdownLang` to avoid a variable name collision with
`let markdown = new MarkdownIt(...)`. Keep this in mind when renaming variables.

### markdown-it-emoji Import

Starting with v3, the default export was removed in favor of a named export.

```js
// Correct (v3+)
import { full as emoji } from "markdown-it-emoji";

// Wrong (v2 and earlier)
import emoji from "markdown-it-emoji";
```

### Shared Renderer (render.js)

The renderer logic is shared between `index-core.js` and `index.js` via
`render.js`. It exposes `window._hljs` so the extended-languages bundle can
register additional languages without re-running `initRenderer`.

### APIs Exposed on `window`

Called from the iOS Swift side via WKWebView's `callAsyncJavaScript`.

| API | Signature | Description |
|-----|-----------|-------------|
| `window.renderMarkdown` | `(payload: { markdown: string, enableImage?: boolean }) => void` | Receives a payload object and renders Markdown. When `enableImage=false`, images are hidden |
| `window.usePlugin` | `(plugin: MarkdownItPlugin) => void` | Public API to register a markdown-it plugin. Allows the Swift side to dynamically add plugins |
| `window._hljs` | `hljs instance` | Exposed by `render.js` for the extended-languages bundle to register additional languages |

### WKWebView Callback

After rendering, the document height is sent to WKWebView.

```js
window?.webkit?.messageHandlers?.updateHeight?.postMessage(height);
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [keitaoouchi/MarkdownView](https://github.com/keitaoouchi/MarkdownView) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
