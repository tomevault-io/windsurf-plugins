---
trigger: always_on
description: Browser-based multilingual sentence alignment tool.
---

# ParallelTexts

Browser-based multilingual sentence alignment tool.

## Important notes

- If you need to test something with the dev server, check first if it is already running (usually port 3000). If it is, unless absolutely necessary don't kill/restart it — use the existing instance.
- After every edit session, give a full git command (header + body) that can be copy-pasted into the terminal.

## Product Requirements

Full requirements, target users, and feature specs: [`.docs/PRD.md`](.docs/PRD.md)

## Project Goal

ParallelTexts is a **browser-based multilingual sentence alignment tool** aimed at non-technical users. Users upload two EPUB/PDF/TXT books in different languages, the app extracts text, generates multilingual sentence embeddings (ONNX, fully in-browser), then aligns them via Needleman–Wunsch dynamic programming. The result is a readable parallel ebook and a downloadable TSV. No backend ML required.

## Commands

```bash
pnpm dev          # Vite dev server on port 3000
pnpm build        # Vite build, then strips any stray local model files from output
pnpm preview      # Build + serve via Wrangler (mirrors production)
pnpm deploy       # Build + deploy to Cloudflare Workers
pnpm test         # Vitest
pnpm lint         # ESLint (TanStack config) — ALWAYS run after every edit session; fix all errors before continuing
pnpm format       # Prettier (writes)
pnpm typecheck    # tsc --noEmit
```

Run a single test file: `pnpm vitest run src/path/to/file.test.ts`

## Before every edit/session

**Always read `.docs/PRD.md`** before:
- Starting any new task or feature
- Making architectural decisions
- Answering questions about project scope or requirements

The PRD contains the "what" and "why" of this project. Keep it in mind as the primary source of truth for project direction.

## After every edit/session

- Run `pnpm lint` to lint the code.
- After implementing a feature, give me a full git command (git command itself + header in semantic commit format + body), something i can easily copy paste, part of the commit message should be the reason for the change/feature/fix/etc (make the commit message multiline if necessary)



## Architecture

### Stack

- **Framework**: TanStack React Start (SSR) + TanStack Router (file-based routing)
- **Deployment**: Cloudflare Workers (`src/server.ts` entry) + R2 (`src/server/serve-r2-assets.ts`) for serving the sample-book EPUBs
- **Styling**: Tailwind CSS v4, shadcn/ui (Base UI + Maia style), Lucide icons
- **Storage**: Dexie v4 (IndexedDB ORM) — all books and alignments are stored client-side
- **ML**: Transformers.js 4 — runs ONNX sentence-transformer models in the browser via WASM or WebGPU

### Model Registry (`src/utils/model.ts`)

Four models are available (user picks in Settings or in the Align form):

| Label | HF ID | Size | Notes |
|---|---|---|---|
| mpnet multilingual | `Xenova/paraphrase-multilingual-mpnet-base-v2` | 1110 MB | **Recommended**, 50+ langs |
| MiniLM L12 | `Xenova/paraphrase-multilingual-MiniLM-L12-v2` | 470 MB | Fastest, smallest |
| DistilUSE base multilingual v2 | `Xenova/distiluse-base-multilingual-cased-v2` | 539 MB | Well-rounded |
| EmbeddingGemma 300M | `onnx-community/embeddinggemma-300m-ONNX` | 1230 MB | 100+ langs, 2048-token ctx |

`loadExtractor()` is a lazy singleton cached per (modelId, device). `downloadModel()` pre-warms the browser Cache API without touching the singleton.

### Model Serving

Embedding models are fetched directly from the Hugging Face Hub in the browser (`env.allowLocalModels = false` in `src/utils/model.ts`) — no backend involved. `downloadModel()` pre-warms the browser Cache API; `checkModelCached()` scans CacheStorage for a matching cached response (models are cached under the full HF resolve URL).

R2 is unrelated to embedding models — it exists solely to serve the sample alignment EPUBs used by the homepage's `SamplesSection`, via the `/assets/` Worker proxy (`src/server/serve-r2-assets.ts`). Those files are git-ignored (`public/assets/sample_books/`) and uploaded to R2 with `pnpm upload-assets`; the post-build script `scripts/strip-stray-build-assets.mjs` also strips any stray local model dumps (e.g. from manual testing) out of the Vite output so they never ship with the app.

### Data Flow

1. User drops EPUB/PDF/TXT → `src/components/drop-zone.tsx` + `src/lib/epub.ts` / `src/lib/pdf.ts` / `src/lib/txt.ts` extract metadata and the full file blob → stored in Dexie via `src/store/books.ts`
2. On alignment trigger (`src/components/align-books-form.tsx`): text is extracted on-demand, preprocessed (optional regex rules; furigana stripping for Japanese), split into sentences via `src/lib/sentence-splitter.ts`, embedded via `src/utils/model.ts` in a Web Worker (`src/workers/alignment.worker.ts`)
3. Banded Needleman–Wunsch (`src/lib/banded-nw.ts`) aligns sentence pairs globally
4. `AlignmentRecord` (with full `AlignmentResult` + `AlignmentMeta`) is written to Dexie via `src/store/alignments.ts`
5. Alternatively, alignments can be imported from a TSV via `src/lib/import-tsv.ts`

### State Management


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nullspace05/ParallelTexts](https://github.com/nullspace05/ParallelTexts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
