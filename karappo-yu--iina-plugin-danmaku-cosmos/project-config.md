---
trigger: always_on
description: An IINA danmaku plugin supporting Niconico (XML / V1 JSON), Bilibili (XML), and **Dandanplay network danmaku** with dual CSS and Canvas rendering modes. The plugin targets **both danmaku style ecosystems**: the native niconico style (large font, fast scroll, bold) and the Chinese/Bilibili style (small font, slow scroll, thin) — switchable via **Style Presets** in the sidebar, plus fine-grained manual controls (font family/weight, font scale, scroll speed, stroke, time offset).
---

# Danmaku Cosmos — Project Guide

## Overview

An IINA danmaku plugin supporting Niconico (XML / V1 JSON), Bilibili (XML), and **Dandanplay network danmaku** with dual CSS and Canvas rendering modes. The plugin targets **both danmaku style ecosystems**: the native niconico style (large font, fast scroll, bold) and the Chinese/Bilibili style (small font, slow scroll, thin) — switchable via **Style Presets** in the sidebar, plus fine-grained manual controls (font family/weight, font scale, scroll speed, stroke, time offset).

The CSS renderer (a fork addition, `mode: "css"`) renders text via DOM/CSS instead of canvas: small fonts stay crisp (system font pipeline vs canvas bitmap sampling) and scroll animations run on GPU compositing (`transform`), which is smoother than canvas in IINA's WKWebView.

## Reference Links

- **Dandanplay API (Swagger)**: https://api.dandanplay.net/swagger/index.html#/
- **IINA plugin API docs**: https://docs.iina.io/index.html

## Tech Constraints

- **IINA plugin environment**: No build tools, bundlers, or npm package managers
- **Rendering engine**: IINA uses Safari (WebKit) internally
- **Language**: Plain vanilla JavaScript (ES5/ES6 mixed), no TypeScript
- **Modularity**: Overlay files are loaded via `<script>` tags in order, sharing global functions and variables through the `window` object. `main.js` (plugin entry) and `sidebar/` run in separate contexts.
- **Network access**: Requires `permissions: ["network-request"]` and `allowedDomains` in `Info.json`. Use `iina.http` module (not browser `fetch`).

## Project Structure

```
Danmaku Cosmos/
├── Info.json                 # Plugin metadata & preference defaults
├── main.js                   # Plugin entry: IINA API, file loading, message relay, DDP integration
├── global.js                 # Global entry (logging only)
├── overlay/                  # Danmaku render layer (WebView container)
│   ├── index.html            # Entry point
│   ├── index.css             # Render styles
│   ├── input.js              # Danmaku data parser (Niconico XML, Bilibili XML)
│   ├── main.js               # Engine entry: message handling, render mode, state mgmt
│   └── lib/                  # Third-party libs (read-only, do not modify)
│       ├── niconicomments.min.js  # Forked niconicomments with CSSRenderer
│       └── opencc.min.js          # opencc-js v1.4.1 UMD bundle (繁→简; LAZY-loaded on first force-simplified enable, not in index.html). Conversion only applies to formatted-path types (nico-xml, bilibili-xml, dandanplay); nico-json is Japanese-only and never converted
├── sidebar/                  # IINA sidebar control panel
│   ├── index.html            # Layout (general settings incl. Style Preset; advanced settings)
│   ├── index.css
│   └── index.js              # UI logic, STYLE_PRESETS, i18n dict (en/ja/zh, ja/zh as \uXXXX escapes); main.js has its own PLUGIN_I18N dict for OSD/menu strings
├── png/                      # README screenshots (excluded from release package by release.yml)
└── .github/workflows/        # Release packaging
    └── release.yml
```

## Dual Rendering Architecture

### CSS Mode (niconicomments CSSRenderer) — default

When `mode: "css"` is passed to NiconiComments, a `CSSRenderer` is created instead of using canvas drawing. The CSSRenderer:

- Creates a 16:9 aspect ratio container (`[data-dm-css-container]`) centered in the viewport
- Uses `--dm-unit` CSS custom property (`min(100vh, 56.25vw) / 1080`) for responsive coordinate mapping
- Renders each danmaku as a `div[data-dm-comment]` with `will-change: transform, opacity; contain: layout style`
- Scroll danmaku: CSS `@keyframes dm-scroll` driven by JS-computed duration (inline `animation` style), matching `getPosX()` formula
- Fixed danmaku (ue/shita): CSS `@keyframes dm-fade` animation
- Stroke: `-webkit-text-stroke` + `paint-order: stroke fill`
- Object pool (max 512 elements) for DOM reuse
- Pause/resume via Web Animations API `pause()`/`play()` (container class-driven)
- Tracks reverse state per danmaku, reanimates when `@reverse` activates/deactivates

### Fonts config (CRITICAL — validation trap)

Fonts are passed to the engine via init config `config.fonts`. **The engine's `isValidFonts` check (typeGuard) requires BOTH `flash` and `html5` key groups** — passing only `{ html5 }` makes the constructor throw `InvalidOptionError` and **all danmaku disappear** (verified in production). Build via `buildDanmakuFontConfig()` in `overlay/main.js`:

```js
{
  flash: { gulim: 'normal 600 [size]px gulim, <stack>, Arial', simsun: '...' },  // Flash mode removed; keep template for validation
  html5: {
    gothic: { font: <family>, offset: <n>, weight: <n> },   // offsets must be within ±1024
    mincho: { font: <family>, offset: <n>, weight: <n> },
    defont: { font: <family>, offset: <n>, weight: <n> },
  },
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [karappo-yu/iina-plugin-danmaku-cosmos](https://github.com/karappo-yu/iina-plugin-danmaku-cosmos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
