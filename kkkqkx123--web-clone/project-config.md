---
trigger: always_on
description: Always use English in code files(include config files, comments) and use Simplified Chinese in docs.
---

# web-clone

**Language**
Always use English in code files(include config files, comments) and use Simplified Chinese in docs.

## Project Overview

**web-clone** is a single-execution web page snapshot tool that downloads and bundles a complete webpage snapshot into either a self-contained HTML file or a directory structure. It can optionally extract and analyze component structure.

**Core capabilities:**
- Snapshot: Download entire webpage (HTML, CSS, JS, images, fonts, media)
- Output: Single HTML file or directory bundle with separated assets
- Transform: Extract component structure with state/event analysis (optional)

## Build & Development

```bash
npm run build         # TypeScript → dist/ (dist/cli.js is the binary)
npm run dev           # Run via tsx without compilation
npm run snapshot      # Alias for dev
```

Entry point for development: `src/cli.ts`

## CLI Usage

```bash
npm run dev -- <url> [options]
npx tsx src/cli.ts <url> [options]
node dist/cli.js <url> [options]  # After npm run build
```

**Common options:**
- `-o, --output <path>` — Output path (default: `./snapshot`)
- `-m, --mode <type>` — `single` (HTML file) or `bundle` (directory, default)
- `--extract-components` — Extract component structure (works with any mode)
- `--framework <hint>` — Framework hint for component extraction: `vue`, `react`, or `svelte`
- `--component-depth <n>` — Component recognition depth threshold (default: 4)
- `--max-assets <n>` — Limit total assets (default: 100)
- `--concurrency <n>` — Parallel downloads (default: 6)
- `--timeout <ms>` — Per-resource timeout (default: 15000)
- `--no-inline` — Skip data URI inlining
- `--pretty` — Prettify HTML
- `--skip-types <extensions>` — Comma-separated extensions to skip (e.g. `.zip,.mp4,.ts`); empty to disable; defaults to archives/installers/docs/video/audio/binaries
- `--max-file-size <size>` — Hard size limit per file (e.g. `50MB`, `10m`, or bytes; default: 50MB; 0 = no limit)

## Architecture

The snapshot workflow is orchestrated by `assembler.ts` in these stages:

### Main Pipeline (Snapshot)

1. **Fetch HTML** (`fetcher.ts:fetchWithTimeout`) — Fetch page with timeout and User-Agent header
2. **Parse HTML** (`parser/html-parser.ts:parseHtml`) — Extract asset refs (CSS, JS, img, font, media)
3. **Recursive CSS extraction** (`assembler.ts` → `parser/css-parser.ts`) — Download external CSS files, extract nested assets
4. **Deduplicate** (`assembler.ts:dedupe`) — Remove duplicate URLs
5. **Download assets** (`fetcher.ts:downloadAllAssets`) — Concurrent workers with retry and validation
6. **Assemble output**:
   - **Bundle mode** (`output/bundle.ts:assembleBundle`) — Write assets to `assets/{css,js,img,fonts,data}/`, rewrite HTML paths
   - **Single mode** (`output/single-file.ts:assembleSingleFile`) — Inline all CSS/JS, convert images/fonts to data URIs

### Optional: Component Extraction Pipeline

When `--extract-components` is specified:

1. **Extract inline CSS/JS** (`assembler.ts:extractInlineCss/extractInlineJs`) — From `<style>` and `<script>` tags
2. **Merge with downloaded assets** (`assembler.ts:extractCssFromAssets/extractJsFromAssets`) — Combine with downloaded CSS/JS
3. **HTML Analysis** (`transform/component-analyzer.ts:enhanceHtmlAnalysis`)
   - Identify component boundaries: explicit markers → semantic tags → (optionally) depth-based
   - **No implicit depth limit**: By default, all DOM depths are analyzed for component boundaries
   - **Optional depth constraint**: Use `--component-depth <n>` to limit recognition to specified depth
   - Extract dynamic points: bindings, events, conditions
   - Build component hierarchy
4. **CSS Analysis** (`transform/css-analyzer.ts:enhanceCssAnalysis`)
   - Extract CSS variables
   - Group rules by component (BEM-based)
   - Mark dynamic styles
5. **JS Analysis** (`transform/js-analyzer.ts:analyzeJavaScript`)
   - Extract state variables (heuristic-based)
   - Identify event handlers and lifecycle hooks
   - Track DOM references
6. **Correlation** (`transform/correlator.ts:correlateComponents`)
   - Match HTML components with CSS rules (class/ID matching)
   - Match with JS logic (DOM ref matching)
   - Calculate match confidence scores
7. **Generation** (`transform/generator.ts:generateComponentStructure`)
   - Build component specs with manifests
   - Estimate migration effort
   - Generate suggestions
8. **Output** (`output/convert.ts:assembleConvert`)
   - Write component directories
   - Generate README, MIGRATION guide
   - **NEW**: Generate REVIEW_REQUIRED.md for low-confidence components

### Key Modules

- **`types.ts`** — Shared types: `SnapshotOptions`, `Asset`, `AssetRef`, `ComponentSpec`, etc.
- **`fetcher.ts`** — HTTP fetching with AbortController timeout, concurrent worker pool, retry logic
- **`validators.ts`** — MIME validation, file extension→MIME mapping, content integrity checks
- **`parser/url-resolver.ts`** — URL resolution (relative→absolute), srcset parsing
- **`parser/css-parser.ts`** — CSS tree parsing for `@import`, `url()` extraction
- **`transform/*`** — Component analysis and correlation engines
- **`cli.ts`** — Commander CLI with orthogonal options design

### Data Structures

```typescript
// CLI input - unified options

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kkkqkx123/web-clone](https://github.com/kkkqkx123/web-clone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
