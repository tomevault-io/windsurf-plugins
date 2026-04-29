---
trigger: always_on
description: MultAI is a **Chrome extension** (Manifest V3) that embeds multiple AI chat providers (ChatGPT, Claude, Gemini, Grok, Meta AI, DeepSeek, Qwen) as iframes inside a single "cockpit" tab. Users type one prompt and broadcast it to all active providers simultaneously. There is **no backend server** — each iframe runs the real provider site under the user's own login, and all data stays in `chrome.storage.local`.
---

# AGENTS.md — MultAI

## Project overview

MultAI is a **Chrome extension** (Manifest V3) that embeds multiple AI chat providers (ChatGPT, Claude, Gemini, Grok, Meta AI, DeepSeek, Qwen) as iframes inside a single "cockpit" tab. Users type one prompt and broadcast it to all active providers simultaneously. There is **no backend server** — each iframe runs the real provider site under the user's own login, and all data stays in `chrome.storage.local`.

## Tech stack

- **Plain JavaScript** (ES modules for extension pages, IIFE for content scripts). No TypeScript.
- **Plain CSS** + Google Fonts. No preprocessors or CSS frameworks.
- **No build step.** Load the repo folder directly as an unpacked Chrome extension.
- **No npm/yarn dependencies.** No `package.json`, no `node_modules`.
- **No CI/CD, linter, or formatter** configured in the repo.

## Architecture

```
┌─────────────────────────────────────────────────────┐
│  Extension pages (chrome-extension://...)            │
│  ┌──────────────┐  ┌────────────┐  ┌─────────────┐  │
│  │ cockpit.js   │  │ options.js │  │ service-     │  │
│  │ (main UI)    │  │ (settings) │  │ worker.js    │  │
│  └──────┬───────┘  └─────┬──────┘  └──────┬───────┘  │
│         │                │                │          │
│         │  chrome.storage.local           │ DNR      │
│         │  ◄─────────────┘         rules ─┘          │
│         │                                            │
│         │  postMessage (multai:*)                    │
│         ▼                                            │
│  ┌──────────────────────────────────┐                │
│  │  Provider iframes                │                │
│  │  anti-framebust.js (MAIN world)  │                │
│  │  _runtime.js + content.js        │                │
│  └──────────────────────────────────┘                │
└─────────────────────────────────────────────────────┘
```

**Embedding flow:** Static + dynamic `declarativeNetRequest` rules strip framing headers (X-Frame-Options, CSP, COOP, COEP, CORP, Permissions-Policy). `anti-framebust.js` runs in MAIN world at `document_start` to neutralize JS-based frame-busting.

**Communication:** Cockpit ↔ provider content scripts talk via `window.postMessage` with `multai:*`-prefixed message types. `sendToPane()` in `src/shared/messaging.js` posts a message and awaits a reply matched by `_replyTo`.

**Readiness:** After an iframe loads, cockpit sends `multai:wake` on an interval until the content script replies `multai:ready`.

## Directory layout

```
manifest.json              Extension manifest (MV3)
rules/iframe-headers.json  Static DNR rules for header stripping
assets/                    Icons (icon.svg; PNGs referenced but may be missing)
src/
  background/
    service-worker.js      DNR setup, action click handler, tab management
  cockpit/
    cockpit.html/css/js    Main UI: pane grid, broadcast, compare, bench, library
  options/
    options.html/css/js    Settings page: crew toggles, plan display, storage reset
  shared/
    providers.js           Provider registry (ids, labels, URLs, origins)
    messaging.js           MSG constants, sendToPane() helper
    attachments.js         File size/name utilities
    anti-framebust.js      MAIN-world frame-busting countermeasure
    design-tokens.css      Shared CSS variables
  providers/
    _runtime.js            Shared content-script framework (register(), DOM helpers)
    chatgpt/content.js     ChatGPT automation (standalone, does not use _runtime)
    claude/content.js      Claude automation (uses _runtime.register)
    gemini/content.js      Gemini automation
    grok/content.js        Grok automation
    meta/content.js        Meta AI automation
    deepseek/content.js    DeepSeek automation
    qwen/content.js        Qwen automation
```

## Key modules

| Module | Responsibility |
|--------|---------------|
| `src/shared/providers.js` | Single source of truth for provider metadata: id, label, default URL, temporary-chat URL, `postMessage` origin. Exports `DEFAULT_CREW`. |
| `src/shared/messaging.js` | Message protocol constants (`MSG.WAKE`, `MSG.BROADCAST`, etc.) and `sendToPane()` with timeout support. |
| `src/cockpit/cockpit.js` | Main UI controller. Manages pane lifecycle, layout (grid/tabs), broadcast dispatch, Compare drawer, prompt library, bench, keyboard shortcuts. State persists via `chrome.storage.local` under `multai.*` keys. |
| `src/providers/_runtime.js` | MAIN-world shared framework. Provides `setPrompt`, `submit`, `attachFiles`, `readLast`, and `register(config)` which wires up the `postMessage` listener for all standard operations. |
| `src/providers/*/content.js` | Per-provider selectors and overrides. All providers call `__multaiRuntime.register({...})`. |
| `src/background/service-worker.js` | Installs dynamic DNR rules, opens/focuses cockpit on action click, handles `multai:open-in-tab`. |

## Judge feature


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Arvid-pku/MultAI](https://github.com/Arvid-pku/MultAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
