---
trigger: always_on
description: > `stream-chain` creates a chain of streams out of regular functions, asynchronous functions, generators, Node streams, and Web streams, with proper per-item backpressure. The default chain returns a Node `Duplex`; subpath variants run natively on Web Streams (`stream-chain/web`) or pure async iterables (`stream-chain/core`). Zero runtime dependencies.
---

# AGENTS.md — stream-chain

> `stream-chain` creates a chain of streams out of regular functions, asynchronous functions, generators, Node streams, and Web streams, with proper per-item backpressure. The default chain returns a Node `Duplex`; subpath variants run natively on Web Streams (`stream-chain/web`) or pure async iterables (`stream-chain/core`). Zero runtime dependencies.

For project structure, module dependencies, and the architecture overview see [ARCHITECTURE.md](./ARCHITECTURE.md).
For detailed usage docs and API references see the [wiki](https://github.com/uhop/stream-chain/wiki).
For migrating from 3.x see [Migration-V3-to-V4](https://github.com/uhop/stream-chain/wiki/Migration-V3-to-V4).

## Setup

This project uses a git submodule for the wiki:

```bash
git clone --recursive https://github.com/uhop/stream-chain.git
cd stream-chain
npm install
```

## Commands

- **Install:** `npm install`
- **Test:** `npm test` (runs `tape6 --flags FO`)
- **Test (Bun):** `npm run test:bun`
- **Test (Deno):** `npm run test:deno`
- **Test (sequential):** `npm run test:seq` (also `test:seq:bun`, `test:seq:deno`)
- **Test (single file):** `node tests/<bucket>/test-<name>.js` (bucket is `core`, `web`, or `node`)
- **TypeScript check:** `npm run ts-check`
- **JavaScript type check (dual tsconfig):** `npm run js-check`
- **TypeScript tests:** `npm run ts-test` (also `ts-test:bun`, `ts-test:deno`)
- **Bench:** `npm run bench -- bench/<name>.js`
- **Lint:** `npm run lint` (Prettier check)
- **Lint fix:** `npm run lint:fix` (Prettier write)

## Project structure

```
stream-chain/
├── package.json                  # Package config; "tape6" section configures test discovery
├── src/                          # Source code
│   ├── index.js                  # /node entry: chain() factory + asStream + asWebStream + gen + re-exports
│   ├── index.d.ts                # TypeScript definitions for the /node public API
│   ├── defs.js                   # Special values (none, stop, many, finalValue, flushable, etc.) + Web/Node stream type guards
│   ├── defs.d.ts                 # TypeScript definitions for defs
│   ├── exec.js                   # Shared sync-when-possible value-or-promise executor (engine behind gen/fun/asStream/asWebStream)
│   ├── exec.d.ts
│   ├── gen.js                    # Push→pull async-generator bridge over exec
│   ├── gen.d.ts
│   ├── fun.js                    # Creates function pipeline from functions (sync-first; collects via exec.next; exported via /web and /core)
│   ├── fun.d.ts
│   ├── dataSource.js             # Coerces a function or iterable to an iterator-producing function (substrate-agnostic)
│   ├── dataSource.d.ts
│   ├── asStream.js               # Wraps a function as a Node Duplex with per-item backpressure
│   ├── asStream.d.ts
│   ├── asWebStream.js            # Wraps a function as a Web Streams {readable, writable} duplex pair
│   ├── asWebStream.d.ts
│   ├── typed-streams.js          # TypeScript helpers: TypedReadable, TypedWritable, TypedDuplex, TypedTransform
│   ├── typed-streams.d.ts
│   ├── node/                     # Subpath: stream-chain/node — canonical Node Streams chain (re-export of root)
│   │   ├── index.js
│   │   └── index.d.ts
│   ├── web/                      # Subpath: stream-chain/web — native Web Streams chain (no node:stream)
│   │   ├── index.js              # chain() over {readable, writable} duplex pairs
│   │   └── index.d.ts
│   ├── core/                     # Subpath: stream-chain/core — substrate-free async-iterable chain
│   │   ├── index.js              # chain() returning a callable async-generator factory
│   │   └── index.d.ts
│   ├── jsonl/                    # JSONL (line-separated JSON) support
│   │   ├── parser.js             # JSONL parser function (returns gen() pipeline)
│   │   ├── parser.d.ts
│   │   ├── parserStream.js       # JSONL parser as a Node Duplex
│   │   ├── parserStream.d.ts
│   │   ├── parserWebStream.js    # JSONL parser as a Web Streams duplex pair
│   │   ├── parserWebStream.d.ts
│   │   ├── stringerStream.js     # JSONL stringer as a Node Transform
│   │   ├── stringerStream.d.ts
│   │   ├── stringerWebStream.js  # JSONL stringer as a Web Streams TransformStream
│   │   └── stringerWebStream.d.ts
│   └── utils/                    # Utility functions
│       ├── take.js               # Take N items from stream
│       ├── takeWhile.js          # Take items while condition is true
│       ├── takeWithSkip.js       # Skip then take
│       ├── skip.js               # Skip N items
│       ├── skipWhile.js          # Skip items while condition is true
│       ├── fold.js               # Reduce/fold stream to single value
│       ├── reduce.js             # Alias for fold
│       ├── scan.js               # Running accumulator (like fold but emits each step)
│       ├── batch.js              # Group items into fixed-size arrays
│       ├── readableFrom.js       # Convert iterable to Node Readable stream

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uhop/stream-chain](https://github.com/uhop/stream-chain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
