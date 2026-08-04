---
trigger: always_on
description: hwpilot is a native HWP/HWPX document editor CLI for AI agents. It provides programmatic access to read and write Korean word processor documents.
---

# hwpilot — Development Guide

## Overview

hwpilot is a native HWP/HWPX document editor CLI for AI agents. It provides programmatic access to read and write Korean word processor documents.

## TypeScript Execution Model

### Development (Bun)
During development, run TypeScript directly with Bun:
```bash
bun src/cli.ts <command> [options]
```

Bun handles TypeScript compilation on-the-fly, enabling fast iteration.

### Production (Node.js)
For distribution, compile to JavaScript and run with Node.js:
```bash
bun run build
node dist/src/cli.js <command> [options]
```

The build pipeline:
1. `tsc` compiles TypeScript to JavaScript in `dist/`
2. `tsc-alias` resolves path aliases (`@/*` → `src/*`)
3. `postbuild.ts` replaces shebangs (`#!/usr/bin/env bun` → `#!/usr/bin/env node`)
4. `prepublish.ts` rewrites bin paths in package.json for npm publishing

## Project Structure

```
src/
├── cli.ts                 # CLI entry point
├── types.ts               # Shared type definitions
├── commands/              # Command implementations
│   ├── convert.ts         # HWP → HWPX conversion
│   ├── create.ts          # New document creation
│   ├── edit-format.ts     # Character formatting
│   ├── edit-text.ts       # Text editing
│   ├── find.ts            # Text search
│   ├── image.ts           # Image operations
│   ├── paragraph.ts       # Paragraph addition
│   ├── read.ts            # Document reading
│   ├── table.ts           # Table operations
│   ├── text.ts            # Text extraction
│   └── validate.ts        # File validation
├── daemon/                # Persistent daemon for batch operations
│   ├── client.ts          # Client-side daemon connection
│   ├── dispatch.ts        # Command dispatch
│   ├── entry.ts           # Daemon entry point
│   ├── flush.ts           # Write-back/flush logic
│   ├── holder-hwp.ts      # HWP file holder
│   ├── holder-hwpx.ts     # HWPX file holder
│   ├── launcher.ts        # Daemon process launcher
│   ├── protocol.ts        # Client-server protocol
│   ├── server.ts          # Daemon server
│   └── state-file.ts      # Daemon state persistence
├── formats/
│   ├── hwpx/              # HWPX format (ZIP+XML)
│   │   ├── elements.ts    # XML element definitions
│   │   ├── header-parser.ts # Document header parsing
│   │   ├── loader.ts      # HWPX file loading
│   │   ├── mutator.ts     # HWPX document mutation
│   │   ├── namespaces.ts  # XML namespace handling
│   │   ├── paths.ts       # Internal ZIP paths
│   │   ├── section-parser.ts # Section content parsing
│   │   └── writer.ts      # HWPX file writing
│   └── hwp/               # HWP 5.0 format (binary CFB)
│       ├── cfb-writer.ts  # CFB container writing
│       ├── control-id.ts  # Control character IDs
│       ├── creator.ts     # New HWP file creation
│       ├── mutator.ts     # HWP record mutation
│       ├── reader.ts      # HWP file reading
│       ├── record-parser.ts # Binary record parsing
│       ├── record-serializer.ts # Binary record serialization
│       ├── stream-util.ts # Stream utilities
│       ├── tag-ids.ts     # HWP tag ID constants
│       ├── validator.ts    # HWP structural validation
│       └── writer.ts      # HWP file writing
└── shared/                # Shared utilities
    ├── document-ops.ts    # Document read/write operations
    ├── edit-types.ts      # Edit operation types
    ├── error-handler.ts   # Error formatting
    ├── format-detector.ts # Magic-byte format detection
    ├── output.ts          # JSON output formatting
    ├── ref-hints.ts       # Reference hint generation
    ├── refs.ts            # Reference system (s0.p0.t1...)
    └── viewer.ts          # Hancom Viewer integration

scripts/
├── postbuild.ts           # Post-build shebang replacement
└── prepublish.ts          # Pre-publish bin path rewriting

skills/hwpilot/            # Agent skill definition
└── SKILL.md

.claude-plugin/            # Claude plugin metadata

playground/                # Manual testing with real HWP files
```

## Build Pipeline

### Development Build
```bash
bun run typecheck    # Type-check without emitting
bun run lint         # Lint with oxlint
```

### Production Build
```bash
bun run build        # Compile + alias resolution + postbuild
```

Output: `dist/src/cli.js` (executable with Node.js)

### Publishing
```bash
bun run prepublishOnly   # Build + typecheck + test + rewrite bin paths
npm publish
bun run postpublish      # Restore package.json
```

## Test Commands

```bash
bun test src/        # Run all unit tests
bun test e2e/        # Run all E2E tests
bun run typecheck    # Type-check
bun run lint         # Lint
bun run lint:fix     # Auto-fix lint issues
bun run format       # Format code
```

## E2E Testing

### Philosophy
E2E tests are real-world — they use actual Korean legal documents, invoke the CLI as a subprocess (not function imports), and cross-validate through independent code paths. Unit tests test particular features in controlled environments; E2E tests simulate what an AI agent would actually do with real documents.

### Structure

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devxoul/hwpilot](https://github.com/devxoul/hwpilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
