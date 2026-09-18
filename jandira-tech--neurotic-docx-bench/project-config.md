---
trigger: always_on
description: Guidance for AI agents and contributors working on this repo. Read this before changing
---

# AGENTS.md — neurotic-docx-bench

Guidance for AI agents and contributors working on this repo. Read this before changing
anything; the invariants below are load-bearing.

Before changing, rebuilding, or benchmarking the Rust redline engine, also read
`../reconciliation_plan/GET_JUBARTE_RUST.md`. The only canonical local Rust source
checkout is `../jubarte-redlines` (`~/T/jubarte-redlines`). Everything under this
repository's `src/neurotic_docx_bench/utils/jubarte/jubarte-rust` and
`src/neurotic_docx_bench/utils/jubarte/jubarte-wasm/pkg` is a consumer artifact;
never implement an engine fix in those copies.

EXCEPT IF REQUESTED BY THE PERSON OR REQUIRED BY A SPECIFIC BENCHMARK OR TO DEVELOP A TOOL, YOU MUST RECORD THE RESULTS WITH RASTERS DELETED AFTER EACH TOOL SO SCORING DOESN'T FILL THE DISK.

## What this is

A benchmark that measures how faithfully DOCX tools reproduce **Microsoft Word's**
tracked-change redlines. For each `base → next` document pair, a tool generates a redline
DOCX; the bench renders it to PDF, rasterises every page, and scores it pixel-wise against
a committed **Word oracle** redline for that pair.

## Quickstart

```bash
uv sync                       # Python bench (Python 3.14; needs LibreOffice on PATH)
bun install --frozen-lockfile # Node redline generators (jubarte, docxodus, docx-redline-js)
cd src/neurotic_docx_bench/utils/docxodus && bun install --frozen-lockfile
cd src/neurotic_docx_bench/utils/docx-redline-js && bun install --frozen-lockfile
cd src/neurotic_docx_bench/utils/folio && bun install --frozen-lockfile
cd src/neurotic_docx_bench/utils/superdoc && bun install --frozen-lockfile
uv run bench run              # all tools sequentially → results/bench.jsonl
uv run bench run --only jubarte-final-lossless --limit 5   # one tool, quick
uv run pytest -q              # 68 tests
bunx vitest run               # 7 TS-driver tests (scoped by vitest.config.ts)
```

## Architecture (data flow)

```
bench.yaml → cli.run → per tool:
  resolve tool_version → generate (candidate DOCX) → render (→ PDF) →
  raster (→ PNG per page) → match to oracle by <base>_<next> → score_document →
  append one JSONL line (scores + failures) → gate vs snapshot
```

Key modules (`src/neurotic_docx_bench/`):
- `score.py` `diff.py` `raster.py` `report.py` `html_report.py` `utils.py` — the scoring
  core, **lifted verbatim** from superdoc-visual-benchmarks. **Do not edit their logic** —
  `tests/test_parity.py` guards byte-identical scoring against `tests/reference/`.
- `docxide_metrics.py` + `utils/docxide-metrics/` — the **second scorer**, Jaccard / SSIM /
  text-boundary at 150 DPI, **lifted verbatim** from sverrejb/docxide-pdf `tests/common/`
  (Apache-2.0). Same rule: **do not edit the metric logic** — upstream is the authority and
  `tests/test_docxide_metrics_parity.py` requires the same numbers as upstream's own
  `page-metrics` binary, against frozen values in `tests/reference/docxide_page_metrics.json`.
  Only `src/main.rs` (the batch driver) is ours; it rasterizes, scores and then deletes each
  document's rasters before the next, so a 398-document sweep cannot fill the disk.
- `pipeline.py` — match candidate↔oracle redlines by `<base>_<next>` key, rasterise, score.
- `render/` — `soffice` (LibreOffice, default), `passthrough` (score existing PDFs),
  `playwright` (selector-driven web-editor render), `word` (local-only AppleScript).
- `emit/` — `jsonl` (append-only trend log), `snapshot`, `markdown`, `html`, `gallery`
  (per-run `report.html`: worst-first candidate-vs-oracle page gallery from the
  persisted `score/` rasters; emitted automatically by `bench run`).
- `aggregate.py` `gate.py` `tool_updater.py` `provenance.py` `config.py` `cli.py`.
- `superdoc_gen.py` — the SuperDoc (Python SDK) redline generator.
- `scripts/generate-native-redlines.ts` — jubarte / docxodus / docx-redline-js generators.

## The oracle — READ THIS

- Ground truth: `corpus/word_based/pdf_redlines_word/*.pdf`, named `<base>_<next>_redline.pdf`.
  The tracked-change **markup** is Microsoft Word's; the PDF **rendering** is
  **LibreOffice 26.2.4.2** (`Producer` metadata). Candidates are rendered the same way, so a
  score isolates *redline-markup fidelity vs Word*, not renderer drift.
- Rendering the oracle's own source DOCX via LibreOffice 26.2.4.2 reproduces it
  **pixel-for-pixel → 100** (the `word-redlines-soffice` sanity run). The bench is therefore
  **pinned to LibreOffice 26.2.4.2**; CI regenerates the oracle in-image so any LO version
  works there (see `.github/workflows/bench.yml`).
- `pdf_redlines_word/` also holds ~163 **non-redline base PDFs**; matching excludes them
  (`pipeline.is_redline`) and **raises on any key collision** — never silent last-wins.
- The authoritative pairing is `corpus/word_based/centralized_mapping.csv`
  (`base`, `next`, `pdf_redline = <base>_<next>_redline.pdf`, …).

## Tools benchmarked

| Run | Engine | Version source |
|---|---|---|
| `jubarte-final-native` | jubarte `redlineDocx` (CriticMarkup) | `dist/jubarte-final` content-hash |
| `jubarte-final-lossless` | jubarte `compareDocx` (its in-tree docxodus port) | `dist/jubarte-final` content-hash |
| `docxodus` | real JSv4/docxodus WASM `compareDocuments` | npm `docxodus@9.8.0` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jandira-tech/neurotic_docx_bench](https://github.com/jandira-tech/neurotic_docx_bench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
