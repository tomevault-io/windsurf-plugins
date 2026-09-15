---
trigger: always_on
description: This file provides guidance to Claude Code when working with this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project Overview

**TRANSLATE!** is a browser extension for local-first translation across Chrome (MV3) and Firefox (MV2). The shipped paths are Chrome Built-in (native, Chrome 138+), OPUS-MT (stable downloaded local baseline), TranslateGemma (experimental accelerated local path via offscreen WebGPU/WebNN at `src/offscreen/translategemma.ts`), and optional cloud providers (DeepL, OpenAI, Anthropic, Google Cloud). NLLB-200 is in tree at `src/providers/nllb-200.ts` as an opt-in research path. PR #509 shipped the content-script WebMCP surface, which exposes the extension as MCP tools (`translate_page`, `translate_selection`, `detect_language`) via `src/content/webmcp.ts` for in-page agent integrations.

## Architecture (v2.0+)

```
src/
├── types/                 # TypeScript type definitions
├── core/                  # Core translation infrastructure (~25 modules: throttle, circuit-breaker, glossary, language-detector, prediction-engine, translation-cache, translation-router, webgpu-detector, ...)
├── providers/             # 7 translation provider implementations + base abstract
│   ├── base-provider.ts
│   ├── opus-mt-local.ts   # Helsinki-NLP OPUS-MT via Transformers.js
│   ├── chrome-translator.ts # Chrome Built-in (Chrome 138+)
│   ├── deepl.ts
│   ├── openai.ts
│   ├── anthropic.ts
│   ├── google-cloud.ts
│   ├── nllb-200.ts        # opt-in NLLB research path (not exported from index.ts)
│   └── cloud-provider.ts  # shared cloud-provider helpers
├── offscreen/
│   └── translategemma.ts  # experimental WebGPU/WebNN local accelerated path
├── content/
│   ├── index.ts           # main content script (DOM walking, MutationObserver)
│   └── webmcp.ts          # MCP tool surface for in-page agent integrations (~553 LOC, e2e harness in e2e/webmcp-harness.html)
├── popup/                 # Solid.js popup UI
├── options/               # Settings page
├── background/            # MV3 service worker + Firefox MV2 background page
├── manifest.json          # Chrome (MV3)
└── manifest.firefox.json  # Firefox (MV2)
```

The src/ tree is far richer than this skeleton; treat the listing above as the load-bearing surfaces for agent onboarding. For a full file inventory run `fd -t f . src/` rather than asking the doc.

## Canonical shipped runtime paths

### Chrome
- `popup/content -> background service worker -> offscreen document`
  - `opus-mt` (stable downloaded baseline)
  - `translategemma` (experimental, WebGPU/WebNN)
  - cloud providers (`deepl`, `openai`, `anthropic`, `google-cloud`)
- `popup/content -> chrome.scripting.executeScript(...)`
  - `chrome-builtin` (preferred native path when available, Chrome 138+)

### Firefox
- `popup/content -> background-firefox`
  - `opus-mt`
  - `translategemma`
  - configured cloud providers
- `chrome-builtin` is not available on Firefox.

### Guidance
- Treat `chrome-builtin` and `opus-mt` as the canonical shipped user-facing translation paths.
- Treat `translategemma` as experimental.
- Do not treat `translation-router`, `localModel`, `llama.cpp`, or `wllama` surfaces as the canonical shipped runtime unless a task explicitly targets those legacy/experimental paths.
- The content-script WebMCP surface in `src/content/webmcp.ts` is canonical for in-page MCP-aware agent integrations (Claude.ai, agent harnesses). Do not assume agents only reach the extension via the popup.

## Tech Stack

- **Language**: TypeScript (strict mode)
- **UI Framework**: Solid.js
- **Build Tool**: Vite
- **ML Runtime**: Transformers.js with WebGPU/WASM
- **Models**: Helsinki-NLP OPUS-MT (quantized)

## Common Commands

```bash
npm install          # Install dependencies
npm run dev          # Build with watch mode
npm run build        # Chrome production build to dist/
npm run build:firefox # Firefox production build to dist-firefox/
npm run build:all    # Build both Chrome and Firefox
npm run package:firefox # Create Firefox XPI package
npm run typecheck    # TypeScript type checking
npm run test         # Run Vitest tests
```

## Key Features

### Rate Limiting (`src/core/throttle.ts`)
- Sliding window rate limiting
- Exponential backoff with jitter
- Predictive batching for optimal API usage
- Token estimation (~4 chars per token)

### Provider System
- Unified interface via `BaseProvider`
- Strategy-based selection: Smart/Fast/Quality
- Usage tracking and cost monitoring
- WebGPU acceleration when available

### Translation Flow
1. Popup/content sends a typed message to the background runtime
2. Chrome routes local/cloud work through the service worker (and offscreen when needed)
3. Firefox routes work through the persistent background page
4. Native/cloud/local provider execution runs on the selected path
5. Content script replaces DOM text nodes

## Development Notes

### Adding New Providers
1. Extend `BaseProvider` in `src/providers/`
2. Implement `translate()`, `isAvailable()`, `getSupportedLanguages()`
3. Register in `translation-router.ts`

### Testing
- **Chrome**: Load unpacked extension from `dist/` folder
- **Firefox**: Load temporary add-on from `dist-firefox/manifest.json` via `about:debugging`
- Use `test/test-page.html` for manual testing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MikkoParkkola/translate-browser-extension](https://github.com/MikkoParkkola/translate-browser-extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
