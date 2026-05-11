---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Google Apps Script add-on for Google Sheets that provides four tools via a custom menu ("SSI Tools"):
1. **Import Drive Links** — recursively list files from a Drive folder
2. **Extract Text** — pull text from Docs/PDFs/images (OCR via temporary Doc conversion)
3. **Sample Rows** — reproducible dataset sampling with seeded Fisher-Yates shuffle
4. **Run AI** — batch Gemini API inference on selected rows (text or multimodal file mode)

Built with TypeScript, bundled by Rollup into a single IIFE, deployed via clasp.

## Commands

```bash
npm run build               # Clean build to dist/ (rimraf + rollup + copy appsscript.json)
npm run build:watch         # Continuous rebuild on file changes
npm test                    # Run Jest tests
npm run test:watch          # Jest in watch mode
npm run test:coverage       # Run Jest with coverage + enforce per-file thresholds
npm run lint                # ESLint on src/
npm run lint:fix            # ESLint with auto-fix
npm run format              # Prettier on src/ (rewrites files)
npm run format:check        # Check Prettier formatting without modifying files
npm run typecheck           # TypeScript type check without building (server + client tsconfigs)
npm run deploy              # Build + clasp push to HEAD (development)
npm run deploy:watch        # Continuous build + push watch
npm run clasp:open          # Open the Apps Script editor in browser
npm run clasp:logs          # Tail execution logs from Apps Script
npm run clasp:login         # Authenticate clasp (required before first deploy)
```

> **Note for Claude:** `scripts/release.sh` is a human-only operation and must never be invoked by Claude. It is enforced via a deny rule in `.claude/settings.local.json`.

Run a single test file: `npx jest __tests__/utils.test.ts`
Run a single test by name: `npx jest -t "extractId"`

## Architecture

### Build Pipeline

The build produces two outputs via a `rollup.config.js` array:

**Config 1 — Server bundle:**

`src/server/index.ts` → Rollup (IIFE format, assigned to `_GASEntry`) → `dist/index.js`

Apps Script has no module system — it only sees top-level functions in the global scope. Rollup wraps everything in an IIFE assigned to `_GASEntry`, so exports from `index.ts` are not directly visible. The `footer` field in `rollup.config.js` bridges this gap by appending plain global function stubs that delegate into the IIFE:

```js
function onOpen(e) { _GASEntry.onOpen(e); }
function showSidebar() { _GASEntry.showSidebar(); }
// ... one stub per public entry point
```

**To expose a new function to Apps Script, you must do both:**
1. `export` it from `src/server/index.ts`
2. Add a matching global stub in the `footer` of `rollup.config.js`

If you skip step 2, the function will exist in the bundle but Apps Script won't be able to discover or call it.

**If the function is also called from the client, add a third step:**
3. Add it to `src/client/google.d.ts`

`google.d.ts` is hand-maintained — it is not auto-generated from server code. If you skip step 3, the client will typecheck against stale declarations and only fail at runtime.

**Custom functions (callable from spreadsheet cells) require one extra step:**
3. Add a JSDoc comment with `@customfunction` directly on the stub in `rollup.config.js`

The TypeScript-level JSDoc is compiled away by Rollup and does not appear on the global stub. Google Sheets only registers a function as a custom function when `@customfunction` is present in a JSDoc comment on the **global** declaration — the one in the footer. Without it the function executes correctly when called explicitly but does not appear in autocomplete and is not recognized as a custom function by Sheets.

**Config 2 — Client bundle → `dist/Sidebar.html`:**

`src/client/sidebar-entry.ts` → Rollup (IIFE) → `inlineSidebarHtml` plugin → `dist/Sidebar.html`

HtmlService can only serve `.html` files — all JavaScript and CSS must be inlined at build time. The custom `inlineSidebarHtml` Rollup plugin handles this:
1. Compiles `sidebar-entry.ts` to an intermediate JS chunk
2. Reads `src/Sidebar.html` (the template), `src/client/sidebar.css`
3. Replaces `{{STYLES}}` with `<style>…css…</style>` and `{{SCRIPTS}}` with `<script>…js…</script>`
4. Emits `dist/Sidebar.html` as an asset
5. Deletes the intermediate `.js` chunk so clasp never pushes it as a `.gs` file

### Module Dependency Graph

**Server:**
```
src/server/index.ts          (entry point — menu, 4 tool orchestrators, UI handlers, re-exports custom functions)
├── src/server/config.ts         (CONFIG object: API key property name, model name, size limits)
├── src/server/api.ts            (callGeminiAPI, buildGeminiPayload, invokeGemini — pure HTTP adapter via UrlFetchApp;
│                                 buildGeminiPayload resolves ToolId[] via TOOL_REGISTRY, splits grounding vs function tools)
├── src/server/inference.ts      (runInference — unified inference handler for menu-triggered AI calls; no SpreadsheetApp dep;
│                                 returns string|null, null signals caller to skip the row)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [propublica/gas-ssi-toolkit](https://github.com/propublica/gas-ssi-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
