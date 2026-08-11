---
trigger: always_on
description: Core project guidelines for the pdfvision codebase. Apply these rules when working on any code, documentation, or configuration files within the pdfvision project.
---


# pdfvision Project Structure and Overview

## Project Overview

pdfvision is a CLI tool for extracting text, metadata, and page images from PDF files. Designed primarily for AI agents to read PDF contents efficiently. Built in TypeScript/Node.js with pdfjs-dist for PDF processing and @napi-rs/canvas for page rendering. Features content-hash caching under `<os-tmpdir>/pdfvision/` so only the first run per file is slow.

## Directory Structure

```
pdfvision/
├── src/
│   ├── bin/
│   │   └── pdfvision.ts          # Thin CLI entry point (delegates to cli/cli.ts)
│   ├── cli/
│   │   ├── cli.ts            # Argument parsing and dispatch
│   │   ├── help.ts           # Help text (CLI + `pdfvision mcp`)
│   │   ├── mcpCommand.ts     # `pdfvision mcp` subcommand dispatch, resolved before parseArgs
│   │   ├── clearCacheCommand.ts # `pdfvision clear-cache` dispatch and its ambiguity guard
│   │   ├── docsCommand.ts    # `pdfvision docs` dispatch and topic index rendering
│   │   ├── docs/             # Generated topic index + bodies (see scripts/build-cli-topics.mjs)
│   │   ├── optionSpec.ts     # The parseArgs option spec, exported so tests can walk it
│   │   ├── subcommandFlags.ts # --help / --version handling shared by every subcommand
│   │   └── version.ts        # Reads version from package.json
│   ├── mcp/
│   │   ├── serve.ts          # stdio bootstrap behind `pdfvision mcp`
│   │   ├── server.ts         # MCP tool registration (read_pdf / search_pdf / render_pdf)
│   │   ├── tools/            # One module per tool
│   │   ├── source.ts         # Path-or-URL dispatch and SSRF guard
│   │   ├── limits.ts         # Response budgets
│   │   ├── refs.ts           # Short region handles (`p47m1`) for follow-up renders
│   │   ├── attachments.ts    # Embedded-file classification: inline text, image blocks, refuse the rest
│   │   ├── truncate.ts       # Page-boundary clipping with recovery guidance
│   │   └── result.ts         # Content blocks and the untrusted-data banner
│   ├── core/
│   │   ├── processor.ts      # Document-level processing entry point
│   │   ├── annotations/      # PDF annotation normalization and geometry
│   │   ├── document/         # Document-level metadata, outline, layers, attachments
│   │   ├── formFields/       # Form field extraction, labels, and stacked controls
│   │   ├── graphics/         # Image and vector operation analysis
│   │   ├── io/               # OS tmpdir cache and remote input helpers
│   │   ├── layout/           # Text layout reconstruction and repeated chrome
│   │   ├── links/            # Link extraction and visible link text
│   │   ├── ocr/              # OCR dispatch, worker, and word geometry
│   │   ├── options/          # Option parsers such as page ranges
│   │   ├── processor/        # Pipeline helpers used by processor.ts
│   │   ├── quality/          # Page quality and extraction confidence signals
│   │   ├── renderer/         # Per-page PNG rendering and crop handling
│   │   ├── runtime/          # Shared runtime helpers such as bounded parallelism
│   │   ├── search/           # Search query compilation and match geometry
│   │   ├── text/             # Text joining, geometry, spacing, and direction helpers
│   │   ├── visualRegions/    # Figure/table/form visual-region detection
│   │   ├── warningTextOverlap/ # Text-overlap warning geometry
│   │   ├── warnings/         # User-facing warning detection
│   │   └── widgetAppearance/ # Widget appearance stream text extraction
│   ├── output/
│   │   ├── documentMap.ts    # Aggregated map of a document (no page bodies)
│   │   ├── json.ts           # Structured JSON formatter
│   │   ├── markdown.ts       # Agent-readable Markdown formatter
│   │   ├── toon.ts           # TOON formatter
│   │   └── xml.ts            # XML formatter
│   ├── types/
│   │   └── index.ts          # Shared types (DocumentResult, ProcessOptions, ...)
│   └── index.ts              # Library API entry
├── docs/
│   └── cli-topics/           # Source of `pdfvision docs <topic>`; embedded at build time
├── tests/
│   ├── core/                 # Unit and integration tests
│   └── fixtures/sample.pdf   # Hand-crafted minimal PDF
├── .github/workflows/        # ci.yml, npm-publish.yml
├── biome.json
├── tsdown.config.ts
├── vitest.config.ts
└── package.json
```

## Architecture Principles

- **bin/ stays thin**: only sets up error handlers and calls `cli/cli.ts`.
- **cli/ does I/O**: argument parsing, help/version, calling `core/`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yamadashy/pdfvision](https://github.com/yamadashy/pdfvision) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
