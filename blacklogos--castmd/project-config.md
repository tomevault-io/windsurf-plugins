---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Chrome Extension (Manifest V3) that converts webpage HTML to Markdown, extracts heading outlines, and analyzes arbitrary URLs. No build step — files are loaded directly by Chrome.

## Loading & Testing

1. Open `chrome://extensions/`
2. Enable "Developer mode"
3. Click "Load unpacked" → select this directory
4. After code changes, click the reload icon on the extension card

Two automated suites, both `node:test`, no build step:

- `npm test` — pure lib modules (`lib/*.js`) under Node, real DOM via `linkedom`.
- `npm run test:e2e` — the extension loaded unpacked into Chrome for Testing and
  driven with puppeteer: popup UI, content script injection, service worker,
  the local `.md` viewer/editor on real `file://` pages, and the Confluence
  export against a stubbed tenant (real fetch, real JSZip, real download).
  Chrome for Testing is downloaded once into `~/.cache/puppeteer` on first run;
  `CHROME_PATH` overrides it, `CASTMD_E2E_HEADED=1` shows the window. Branded
  Google Chrome cannot be used — it ignores `--load-extension`.

`npm run test:all` runs both. Details and the one deliberate deviation from the
shipped artifact (host permissions, see `tests/e2e/helpers/test-build.js`) are
documented in that helper.

## Architecture

```
manifest.json              Extension config (MV3, permissions, entry points)
popup.html/js              Extension popup UI — orchestrates all user actions
content.js                 Injected into every page — core conversion logic
background.js              Service worker — context menu, keyboard shortcut, badge
md-viewer.js/css           Content script — renders local file:// *.md files as HTML
lib/markdown-to-html.js    Pure MD→HTML renderer (used by md-viewer.js)
lib/html-to-markdown.js    Pure HTML→MD conversion (used by Confluence flow)
lib/confluence-api.js      Confluence Cloud v2 REST client + tree discovery
lib/zip-builder.js         Thin wrapper over JSZip
lib/confluence-export.js   Orchestrator: discover → fetch → convert → ZIP
vendor/jszip.min.js        Vendored JSZip 3.10.1 (MV3 CSP forbids remote scripts)
```

### Message Flow

```
popup.js       ──(chrome.tabs.sendMessage)──►      content.js  (convert, getOutline, getPageMeta)
background.js  ──(chrome.scripting.executeScript)─► content.js  then a clipboard write in the page
```

### Key Design Decisions

- **Content detection** (`findMainContent`): tries semantic HTML selectors in priority order (`main`, `article`, `[role="main"]`, etc.), falls back to content-density analysis (text length minus link text, divided by element count).
- **`shouldSkipElement`**: filters out nav/header/footer/sidebar elements, and anything computed-invisible. Only `content.js` has it — the background worker injects `content.js` as a file rather than a serialized function, so nothing needs to be duplicated for `executeScript` any more.
- **Flat block pass**: `convertToMarkdown` (and `elementToMarkdown` in the lib) query all block tags at once, so `isRenderedByAncestor` decides what an ancestor already covered. `pre`/`table` render their whole subtree via `textContent`, so nothing inside them is emitted again. `p`/`li`/`h1`–`h6` render inline descendants only, so a block nested inside one (a `<pre>` in an `<li>`, a `<table>` in a `<p>`) still gets its own turn — and `inlineNodesToMarkdown` skips `ul,ol,pre,table` subtrees so the same content is not also flattened into the prose. Nested lists are the one exception: `handleLists` recurses into the lists hanging directly off an item. Get this wrong in either direction and the output breaks: too narrow duplicates content mid-line (which demotes the next heading to body text), too wide silently drops code blocks and tables.
- **Filenames** are the popup's job (`slugifyUrl` in `popup.js` for downloads, `HtmlToMarkdown.sanitizeTitle` for ZIP entries). `sanitizeTitle` also has to be traversal-safe: titles become ZIP paths.
- **Confluence tree export** (`lib/confluence-*.js`): runs entirely in popup context. Discovers page tree via Confluence Cloud v2 children API (paginated, BFS, hard cap 100), fetches rendered HTML bodies (`body-format=export_view`) at concurrency 3 with 429 backoff, converts via `lib/html-to-markdown.js`, packages into a ZIP via vendored JSZip, downloads via `<a download>`. MV3 service worker is **not** used — popup-bound execution avoids worker idle-kill on multi-minute jobs. Permission for `https://{tenant}.atlassian.net/*` is requested on-demand at preview time.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [blacklogos/castmd](https://github.com/blacklogos/castmd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
