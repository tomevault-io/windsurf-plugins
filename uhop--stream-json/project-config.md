---
trigger: always_on
description: > `stream-json` is a micro-library of Node.js stream components for creating custom JSON processing pipelines with a minimal memory footprint. It can parse JSON files far exceeding available memory streaming individual primitives using a SAX-inspired API. It depends on [stream-chain](https://www.npmjs.com/package/stream-chain) for pipeline composition.
---

# AGENTS.md — stream-json

> `stream-json` is a micro-library of Node.js stream components for creating custom JSON processing pipelines with a minimal memory footprint. It can parse JSON files far exceeding available memory streaming individual primitives using a SAX-inspired API. It depends on [stream-chain](https://www.npmjs.com/package/stream-chain) for pipeline composition.

For project structure, module dependencies, and the architecture overview see [ARCHITECTURE.md](./ARCHITECTURE.md).
For detailed usage docs and API references see the [wiki](https://github.com/uhop/stream-json/wiki).

## Setup

This project uses a git submodule for the wiki:

```bash
git clone --recursive git@github.com:uhop/stream-json.git
cd stream-json
npm install
```

## Commands

- **Install:** `npm install`
- **Test:** `npm test` (runs `tape6 --flags FO`)
- **Test (Bun):** `npm run test:bun`
- **Test (sequential):** `npm run test:proc`
- **Test (single file):** `node tests/test-<name>.js`
- **TypeScript check:** `npm run ts-check`
- **Bench:** `npm run bench -- bench/<name>.js`
- **Lint:** `npm run lint` (Prettier check)
- **Lint fix:** `npm run lint:fix` (Prettier write)

## Project structure

```
stream-json/
├── package.json          # Package config; "tape6" section configures test discovery
├── src/                  # Source code
│   ├── index.js          # Main entry point: creates Parser + emit()
│   ├── index.d.ts        # TypeScript definitions for the main module
│   ├── parser.js         # Streaming SAX-like JSON parser (token stream)
│   ├── parser.d.ts       # TypeScript definitions for parser
│   ├── assembler.js      # Token stream → JavaScript objects (plain class, `onDone` callback)
│   ├── assembler.d.ts    # TypeScript definitions for assembler
│   ├── disassembler.js   # JavaScript objects → token stream
│   ├── disassembler.d.ts # TypeScript definitions for disassembler
│   ├── stringer.js       # Token stream → JSON text (flushable function + asStream)
│   ├── stringer.d.ts     # TypeScript definitions for stringer
│   ├── emitter.js        # Token stream → EventEmitter events (Writable); .asWebStream → EventTarget
│   ├── emitter.d.ts      # TypeScript definitions for emitter
│   ├── filters/          # Token stream editors
│   │   ├── filter-base.js    # Base for all filters (filterBase + makeStackDiffer)
│   │   ├── pick.js           # Pick subobjects by path
│   │   ├── replace.js        # Replace subobjects with a value
│   │   ├── ignore.js         # Remove subobjects (Replace variant)
│   │   └── filter.js         # Filter tokens preserving shape
│   ├── streamers/        # Token stream → object stream
│   │   ├── stream-base.js    # Base for all streamers (uses Assembler)
│   │   ├── stream-values.js  # Stream successive JSON values
│   │   ├── stream-array.js   # Stream array elements
│   │   └── stream-object.js  # Stream object properties
│   ├── utils/            # Utilities
│   │   ├── emit.js           # Decorate a Node Readable with token events (Web variant at src/web/utils/emit.js → EventTarget)
│   │   ├── with-parser.js    # Create parser + component pipelines
│   │   ├── batch.js          # Batch items into arrays (wraps stream-chain batch)
│   │   ├── verifier.js       # Validate JSON text (gen pipeline + asStream)
│   │   └── flex-assembler.js # Assembler with custom containers (Map, Set, etc.)
│   ├── jsonl/            # JSONL (line-separated JSON) support
│   │   ├── parser.js         # JSONL parser → {key, value} objects
│   │   └── stringer.js       # Objects → JSONL text (Transform stream; .asWebStream → Web TransformStream)
│   ├── jsonc/            # JSONC (JSON with Comments) support
│   │   ├── parser.js         # JSONC parser → token stream (extends parser.js; raw export jsoncParser)
│   │   ├── stringer.js       # JSONC token stream → text (extends stringer.js)
│   │   └── verifier.js       # JSONC validator with error locations (extends verifier.js; raw export jsoncVerifier)
│   ├── file/             # Node-only file I/O (uses node:fs/promises; NOT mirrored in core/ or web/)
│   │   ├── index.js          # Barrel: parseFile, verifyFile, stringerToFile, pipe, drain
│   │   ├── parser.js         # parseFile() — file path → token stream (input-edge stage)
│   │   ├── verifier.js       # verifyFile() — standalone async validator (Promise<void>)
│   │   ├── stringer.js       # stringerToFile() — token stream → file (output-edge sink)
│   │   ├── jsonc/{index,parser,verifier,stringer}.js  # JSONC variants
│   │   └── internal/{block-reader,block-writer}.js     # Shared async-fs primitives
│   ├── core/             # Pure, substrate-agnostic factories (no Node-stream imports)
│   │   ├── utils/{drain,pipe}.js  # New generic helpers — last-value drain + one-shot flush driver
│   │   └── …                 # Mirrors src/ layout: each component's runtime + .d.ts
│   └── web/              # Web Streams substrate entries (browser-safe)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uhop/stream-json](https://github.com/uhop/stream-json) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
