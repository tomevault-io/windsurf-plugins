---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Browser-based PII anonymizer for Polish legal documents. Uses HuggingFace transformer models (bardsai/eu-pii-anonimization variants) for NER, runs entirely client-side via Web Worker. Includes an evaluation framework for measuring detection quality against annotated ground-truth documents.

## Commands

```bash
npm run dev              # Vite dev server
npm run build            # Production build → dist/
npm test                 # Run all tests once (vitest run)
npm run test:watch       # Watch mode
npx vitest run src/pipeline/steps/steps.test.js  # Single test file

# Evaluation (downloads models on first run, takes minutes)
npm run eval             # Process test-data/synthetic/*.txt through pipeline
npm run eval -- --label=my-run  # Tag a run
npm run eval:score       # Compute precision/recall/F1 against .expected.json
npm run eval:compare     # Diff two runs
npm run eval:list        # List available runs

# Performance benchmark (Playwright + real Chromium)
# First-time setup (one-off): downloads ~150MB Chromium binary
npx playwright install chromium

npm run bench                                  # 3 measured runs/case + 1 warmup
npm run bench -- --label=before-fp16           # Tag a run
npm run bench -- --runs=5 --no-warmup          # Adjust runs / skip warmup
npm run bench:list                             # List bench runs
npm run bench:compare latest <run-ref>         # Diff median timings
```

## Architecture

### Pipeline

Core abstraction: a linear pipeline of phases, each containing ordered steps. Every step is an `async (ctx) => ctx` function that receives and returns a context object.

**Context shape** (`src/pipeline/context.js`):
```js
{ text, segments, entities, anonymized, legend, debug }
```

**Phases** (defined in `src/pipeline/configs/default.js`):
1. **preprocess** — normalize whitespace
2. **segment** — split text into sentences (sentencex), chunk long sentences at 900 chars
3. **ner** — run two HF models + regex patterns to extract entities
4. **postprocess** — filter allowed types → snap to word boundaries → filter low-confidence → dedup → merge adjacent → tokenize → rescan for missed PII

The runner (`src/pipeline/runner.js`) threads context through steps and records debug diffs between each step.

### Browser vs Node

- **Browser**: `src/main.js` → `src/worker.js` (Web Worker loads models, runs pipeline)
- **Node** (eval only): `src/eval/run.js` uses `@huggingface/transformers` + `onnxruntime-node` directly

Pipeline steps are environment-agnostic. Model loading is injected via `loadModel` parameter.

### Eval Framework

Ground truth lives in `test-data/synthetic/` as paired `.txt` + `.expected.json` files. Eval runs are stored in `test-data/results/{timestamp}/` with a `latest` symlink. Scoring (`src/eval/score.js`) computes per-document and aggregate precision/recall/F1 using overlap-based entity matching (`src/eval/matching.js`).

### Bench Framework

Playwright drives real Chromium against the Vite dev server, exercising the same UI flow a user runs (paste text → click Analyze → result). Cases are derived at runtime by deduping `ENTITY_SOURCES` arrays (regex stripped) — currently 5 unique source combos plus a 6th "all-entities" baseline. Per case: 1 warmup + 3 measured runs in fresh tabs, sharing IndexedDB cache so models stay warm.

Per case captured: E2E (Playwright wall clock), load (worker `model:load:start`/`end` events summed), inference (classify wall − total load). Static `sizeMB` from `SOURCES`.

Worker emits `{type:'timing', mark, alias?, t}` postMessages. `src/main.js` mirrors them to `console.log` with `[bench-timing]` prefix; `bench/runner.js` parses console events. No bench-only code in production.

Test doc: `test-data/bench/single-page.txt` (~2700 chars with all entity types). Results: `test-data/bench-results/{run-id}/summary.json` with `latest` symlink.

**v1 limitations**: download time excluded (cache-warm only); no fp16 / WebNN support yet (matrix auto-extends when `SOURCES` grows). Use `--no-warmup` to skip the warmup pass when iterating. **Cross-machine variance is unaddressed** — `bench:compare` is meaningful only between runs on the same hardware (CPU, RAM, dtype-acceleration support). Tag the host in `--label=` if you'll compare across machines.

## Conventions

- Pure ESM (`"type": "module"` in package.json)
- Vitest with `globals: true` — no imports needed for `describe`/`it`/`expect`
- No TypeScript, no linter configured
- Pipeline steps are named functions (name appears in debug output)
- Entity format: `{ entity_group, start, end, score, word }`
- **Always tag eval runs** with `--label=<short-descriptive-slug>` (e.g. `npm run eval -- --label=trim-trailing-dot`) so `eval:list` stays navigable and `eval:compare` is meaningful.
- Run `eval` (tagged) and `eval:score` after modifying `src/pipeline`

## WebMCP Integration


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wjarka/pii-anonymizer](https://github.com/wjarka/pii-anonymizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
