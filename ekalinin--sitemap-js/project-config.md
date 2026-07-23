---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

sitemap.js is a TypeScript library and CLI tool for generating sitemap XML files compliant with the sitemaps.org protocol. It supports streaming large datasets, handles sitemap indexes for >50k URLs, and includes parsers for reading existing sitemaps.

## Development Commands

### Building
```bash
npm run build                 # Compile TypeScript to dist/esm/ and dist/cjs/
npm run build:esm             # Build ESM only (dist/esm/)
npm run build:cjs             # Build CJS only (dist/cjs/)
```

### Testing
```bash
npm test                      # Run Jest tests with coverage
npm run test:full             # Run lint, build, Jest, and xmllint validation
npm run test:typecheck        # Type check only (tsc)
npm run test:perf             # Run performance tests (tests/perf.mjs)
npm run test:xmllint          # Validate XML schema (requires xmllint)
```

### Linting
```bash
npx eslint lib/* ./cli.ts     # Lint TypeScript files
npx eslint lib/* ./cli.ts --fix  # Auto-fix linting issues
```

### Running CLI Locally
```bash
node dist/esm/cli.js < urls.txt   # Run CLI from built dist
./dist/esm/cli.js --version       # Run directly (has shebang)
npm link && sitemap --version     # Link and test as global command
```

## Code Architecture

### Entry Points
- **[index.ts](index.ts)**: Main library entry point, exports all public APIs
- **[cli.ts](cli.ts)**: Command-line interface for generating/parsing sitemaps

### File Organization & Responsibilities

The library follows a strict separation of concerns. Each file has a specific purpose:

**Core Infrastructure:**
- **[lib/types.ts](lib/types.ts)**: ALL TypeScript type definitions, interfaces, and enums. NO implementation code.
- **[lib/constants.ts](lib/constants.ts)**: Single source of truth for all shared constants (limits, regexes, defaults).
- **[lib/validation.ts](lib/validation.ts)**: ALL validation logic, type guards, and validators centralized here.
- **[lib/utils.ts](lib/utils.ts)**: Stream utilities, URL normalization, and general helper functions.
- **[lib/errors.ts](lib/errors.ts)**: Custom error class definitions.
- **[lib/sitemap-xml.ts](lib/sitemap-xml.ts)**: Low-level XML generation utilities (text escaping, tag building).

**Stream Processing:**
- **[lib/sitemap-stream.ts](lib/sitemap-stream.ts)**: Main transform stream for URL → sitemap XML.
- **[lib/sitemap-item-stream.ts](lib/sitemap-item-stream.ts)**: Lower-level stream for sitemap item → XML elements.
- **[lib/sitemap-index-stream.ts](lib/sitemap-index-stream.ts)**: Streams for sitemap indexes and multi-file generation.

**Parsers:**
- **[lib/sitemap-parser.ts](lib/sitemap-parser.ts)**: Parses sitemap XML → SitemapItem objects.
- **[lib/sitemap-index-parser.ts](lib/sitemap-index-parser.ts)**: Parses sitemap index XML → IndexItem objects.

**High-Level API:**
- **[lib/sitemap-simple.ts](lib/sitemap-simple.ts)**: Simplified API for common use cases.

### Core Streaming Architecture

The library is built on Node.js Transform streams for memory-efficient processing of large URL lists:

**Stream Chain Flow:**
```
Input → Transform Stream → Output
```

**Key Stream Classes:**

1. **SitemapStream** ([lib/sitemap-stream.ts](lib/sitemap-stream.ts))
   - Core Transform stream that converts `SitemapItemLoose` objects to sitemap XML
   - Handles single sitemaps (up to ~50k URLs)
   - Automatically generates XML namespaces for images, videos, news, xhtml
   - Uses `SitemapItemStream` internally for XML element generation

2. **SitemapAndIndexStream** ([lib/sitemap-index-stream.ts](lib/sitemap-index-stream.ts))
   - Higher-level stream for handling >50k URLs
   - Automatically splits into multiple sitemap files when limit reached
   - Generates sitemap index XML pointing to individual sitemaps
   - Requires `getSitemapStream` callback to create output files

3. **SitemapItemStream** ([lib/sitemap-item-stream.ts](lib/sitemap-item-stream.ts))
   - Low-level Transform stream that converts sitemap items to XML elements
   - Validates and normalizes URLs
   - Handles image, video, news, and link extensions

4. **XMLToSitemapItemStream** ([lib/sitemap-parser.ts](lib/sitemap-parser.ts))
   - Parser that converts sitemap XML back to `SitemapItem` objects
   - Built on SAX parser for streaming large XML files

5. **SitemapIndexStream** ([lib/sitemap-index-stream.ts](lib/sitemap-index-stream.ts))
   - Generates sitemap index XML from a list of sitemap URLs
   - Used for organizing multiple sitemaps

### Type System

**[lib/types.ts](lib/types.ts)** defines the core data structures:

- **SitemapItemLoose**: Flexible input type (accepts strings, objects, arrays for images/videos)
- **SitemapItem**: Strict normalized type (arrays only)
- **ErrorLevel**: Enum controlling validation behavior (SILENT, WARN, THROW)
- **NewsItem**, **Img**, **VideoItem**, **LinkItem**: Extension types for rich sitemap entries
- **IndexItem**: Structure for sitemap index entries
- **StringObj**: Generic object with string keys (used for XML attributes)

### Constants & Limits

**[lib/constants.ts](lib/constants.ts)** is the single source of truth for:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ekalinin/sitemap.js](https://github.com/ekalinin/sitemap.js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
