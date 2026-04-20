---
trigger: always_on
description: **S**mall **I**mage **P**rocessor - Ultra memory-efficient image processing for Cloudflare Workers.
---

# @standardagents/sip

**S**mall **I**mage **P**rocessor - Ultra memory-efficient image processing for Cloudflare Workers.

This package now lives in the standalone `standardagents/sip` repository. It is
published independently and consumed by other Standard Agents packages via npm.

## Why This Exists

Cloudflare Workers: 128MB memory limit. A 25MP JPEG decoded = ~100MB raw pixels. Traditional libraries cause "Network connection lost" errors on large uploads.

**sip solution**: Scanline streaming + DCT scaling = <1MB peak memory for ANY size image.

## Architecture

```
Input (any format) → Decode (scanline) → Resize (2-row buffer) → Encode (scanline) → JPEG output
```

**Memory during 100MP image processing**: ~50KB (not 300MB).

## Format Support

| Format | Decoder | Method | Notes |
|--------|---------|--------|-------|
| **JPEG** | libjpeg-turbo (WASM) | DCT scaling + scanline | Best case: decode at 1/8 scale |
| **PNG** | libspng (WASM) | Row-by-row progressive | Full resolution decode, stream output |
| **WebP** | @jsquash/webp | Full decode | No WASM yet, higher memory |
| **AVIF** | @jsquash/avif | Full decode | No WASM yet, higher memory |

**Output**: Always JPEG (universal, good compression).

## File Structure

```
src/
├── index.ts              # Exports: sip, probe, initStreaming
├── probe.ts              # Format detection from magic bytes (no decode)
├── pipeline.ts           # Main sip.process() orchestration
├── streaming.ts          # WASM streaming processors
├── resize.ts             # Bilinear interpolation (2-row buffer)
├── encoder.ts            # WASM JPEG encoder wrapper
├── types.ts              # Shared types
├── decoders/
│   ├── simple.ts         # @jsquash fallback (WebP/AVIF only)
│   └── types.ts          # Decoder interface
└── wasm/
    ├── index.ts          # WASM module exports
    ├── loader.ts         # WASM loading (browser/Workers/Node)
    ├── decoder.ts        # WasmJpegDecoder class
    ├── png-decoder.ts    # WasmPngDecoder class
    ├── encoder.ts        # WasmJpegEncoder class
    └── types.ts          # WASM function signatures

wasm/
├── build.sh              # Emscripten build script
├── src/sip.c             # C bindings for libjpeg-turbo + libspng
└── libs/                 # Downloaded during build (not committed)
    ├── libjpeg-turbo-3.0.1/
    ├── libspng-0.7.4/
    └── miniz-2.2.0/

docs/
├── index.html            # Static docs shell
├── main.js               # Arrow app entrypoint
├── styles.css            # Docs site styling
└── vite.config.mjs       # Static site build config

scripts/
└── release.mjs           # Local release orchestration for stable and prerelease tags

.github/workflows/
└── publish.yml           # Tag-driven build, npm trusted publish, release notes

docs-dist/                # Generated static site output (gitignored)
```

## Docs Site

This repo ships a static docs site built with Arrow and Vite, plus a markdown
mirror for LLMs.

```bash
pnpm docs:dev
pnpm docs:build
pnpm docs:preview
```

Use it for package-level docs and examples only. Keep it static, client-only, and
independent from the library build. The build output goes to `docs-dist/` and
should not be committed.

### Documentation lives in TWO places — keep them in sync

Whenever you change the API surface, examples, options, or any user-facing
docs content, you MUST update **both**:

1. `docs/main.js` — the interactive Arrow site at <https://sip.standardagents.ai>
2. `docs/public/llms.md` — the LLM-friendly markdown mirror served at `/llms.md`

The "Markdown docs" button in the hero links to `/llms.md`, so users (and LLMs)
will see whatever is in that file. If the two drift, agents will get stale or
contradicting information. Always update both in the same commit.

### SSR + hydration

The docs site is server-rendered at build time and hydrated on the client.

- `docs/main.js` — defines all components and `export`s the `App` factory.
  Module-level browser side effects (fetch, scroll handlers, etc.) are
  guarded behind `if (isBrowser)` so the file can be imported by Node.
- `docs/entry-server.js` — imports `App` and uses
  `renderToString` from `@arrow-js/ssr` to produce an HTML string. Inlines
  the shiki-highlighted code blocks too.
- `docs/scripts/prerender.mjs` — runs after `vite build` completes. Spins
  up Vite in middleware mode, calls `entry-server.renderPage()`, and
  patches the empty `<div id="app"></div>` in `docs-dist/client/index.html`
  with the rendered HTML.
- The client uses `hydrate()` from `@arrow-js/hydrate` instead of `render()`
  so reactive state attaches to the existing server-rendered nodes without
  re-creating them.

When adding new code-block keys to `docs/code-blocks.js`, the prerender step
will automatically inline them — no extra wiring needed. Just don't add
client-only side effects at module load that crash on the server (use the
`isBrowser` guard).

## Release Automation

Publishing is driven by tag pushes. npm releases use trusted publishing from
GitHub Actions, so there is no `NPM_TOKEN` in this repo or workflow.

```bash
pnpm verify              # CI-equivalent check: typecheck, build, docs, unit tests
pnpm verify:release      # Local release gate: rebuild WASM, then run verify

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [standardagents/sip](https://github.com/standardagents/sip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
