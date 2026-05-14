---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A browser-based static web app that visualises the internal bit-level structure of zlib (RFC 1950), gzip (RFC 1952), and raw-DEFLATE (RFC 1951) streams. Three virtualized panes (bytes · structure · decoded output) cross-highlight each other as the user clicks or hovers.

Design spec: `docs/superpowers/specs/2026-04-21-zlib-viz-design.md`.
Original implementation plan: `docs/superpowers/plans/2026-04-21-zlib-viz.md`.

## Commands

```bash
npm run dev              # vite dev server
npm run typecheck        # tsc -b --noEmit   (run this after type-sensitive edits)
npm test                 # vitest run        (68 unit/integration/property tests)
npm test -- test/parser/bit-reader.test.ts    # single file
npm run test:watch       # vitest watch mode
npm run e2e              # Playwright smoke suite (launches its own dev server on :5173)
npm run build            # tsc -b && vite build → dist/

# Regenerate binary fixtures from source strings (after touching generate.ts).
# Remember to copy the gzip example to public/ as .bin (see "gzip gotcha" below):
npx tsx test/parser/fixtures/generate.ts
```

Tests use Vitest (not Jest). Property tests use fast-check with ~280 random inputs per format; parser output is compared byte-for-byte against Node's native `zlib`.

## Architecture — big picture

Four layers, each with one responsibility and a clear interface:

```
UI (React) ──selection/hover──▶ State (Zustand)
                                  │
                                  ▼
                        ParsedStream (immutable)
                                  ▲
                                  │ postMessage (structured clone)
                        Worker (parse-worker.ts)
                                  │
                                  ▼
                        Parser core (pure TS, no DOM)
```

### Single-buffer, bit-addressed model

Everything the UI highlights is expressed in **bit coordinates from `input[0]`**, via `BitRange = { start, end }`. That single convention is what makes cross-pane highlighting work without per-pane stream-walking logic. Every node the parser emits carries a `BitRange` (or enough bit offsets to build one). The parser is eager and single-pass; `ParsedStream` contains the fully decoded output as one `Uint8Array`.

### Parser core (`src/parser/`)

Pure TS, zero DOM imports — reusable from a CLI or tests without the UI. Key invariants:

- **LSB-first bit order.** RFC 1951 defines bits within a byte LSB-first. `BitReader` handles this; `buildHuffmanTable` bit-reverses each canonical code before indexing the flat lookup table.
- **`BitReader.peek(n)` zero-pads past EOF** (but `readBits` still throws). This is required because `decodeSymbol` always peeks `maxBits` ahead, and a valid DEFLATE stream can end with fewer than `maxBits` bits remaining after the end-of-block code; the LSB-first striped lookup resolves the short code correctly regardless of the padded high bits.
- **Error tiers** — fatal (stop parsing the current block; emit `partialParsed`), soft (e.g. ADLER32/CRC32 mismatch; keep going), unexpected (thrown; the worker harness converts to `{ type: 'error' }`).
- **The `dynamic-small.zlib` fixture must actually be dynamic.** Node's zlib picks fixed-Huffman for short inputs; `generate.ts` uses a diverse multi-sentence payload to guarantee BTYPE=2. If you change the input, run the generator and verify with `(bytes[2] >> 1) & 3 === 2`.

### Worker boundary (`src/worker/`)

One fresh worker per parse; input `Uint8Array.buffer` is transferred. The UI keeps its own copy via `new Uint8Array(bytes)` (which is a copy, not a view, when the arg is a TypedArray) and stashes it under `inputBytes` — see Task 18 wiring in `InputArea.onBytes`.

Progress events are currently emitted once per parse (not per block). The plan flags this as a known approximation; if streaming feedback becomes important, thread a callback through `parseDeflate`.

### State (`src/state/`)

One Zustand store (`useUiStore`). Three derived-state patterns to know:

- **`resolveSelection(selection, parsed)`** — the only place that maps a `Selection` to bit/output/backref ranges. Every pane subscribes to its output; panes never walk the stream themselves.
- **`findSelectionAtBit(parsed, bit)`** — reverse direction: clicking a bit resolves to the innermost structural node, drilling into HLIT/HDIST/HCLEN and code-length entries before falling back to symbol / block / wrapper / trailer.
- **`isExpanded(id, expansion)`** — per-row collapse state. Defaults vary by prefix: `block:*` defaults expanded, `symbols:*` defaults collapsed (a block with 10 000 symbols would otherwise drown the tree).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TrueSkrillor/zlib-viz](https://github.com/TrueSkrillor/zlib-viz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
