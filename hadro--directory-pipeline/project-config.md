---
trigger: always_on
description: Extracts structured CSVs from digitized historical directories (city directories, phone books, Green Books) sourced from the Library of Congress, Internet Archive, and any public IIIF endpoint.
---

# directory-pipeline

Extracts structured CSVs from digitized historical directories (city directories, phone books, Green Books) sourced from the Library of Congress, Internet Archive, and any public IIIF endpoint.

## Future direction: local open models

A strategy doc for replacing the Gemini OCR/NER steps with local open models (Chandra, NuExtract3, Qwen3.5, GLiNER) via Hugging Face uv-scripts — plus a cost comparison and a phased roadmap — lives at `docs/plans/huggingface-uv-scripts.md` (created 2026-06-15). Read it before starting any local-model backend work; implementation is deferred to a future session.

## Planned work: automatic section detection

A phased implementation plan for a `detect_sections` stage — auto-drafts `sections.txt` for multi-section city directories (alphabetical / street / classified / frontmatter) so per-section NER prompts route correctly — lives at `docs/plans/section-detection-plan.md` (created 2026-06-15). The consumer side (`utils/section_utils.py` + `extract_entries.py`) already exists; only the producer is missing. Read it before starting section-detection work; best implemented locally where the gitignored `output/` data is on disk.

## Setup

```bash
uv sync                    # core deps
uv sync --extra gpu        # add Surya OCR
uv sync --extra geo        # add geocoding
uv sync --all-extras       # everything
```

Requires `.env` with `GEMINI_API_KEY` (and optionally `GOOGLE_MAPS_API_KEY`).

## Running the pipeline

After `uv sync`, a `pipeline` CLI command is available:

```bash
pipeline run    <URL>          # automated: download → OCR → extract → explore
pipeline guided <URL>          # human-in-loop: pauses for page selection + alignment review
pipeline ingest <URL>          # download only
pipeline calibrate <DIR>       # select sample pages + generate prompts (once per collection type)
pipeline ocr    <DIR>          # Surya OCR + Gemini OCR + align bboxes
pipeline extract <DIR>         # NER extraction + build explorer
pipeline review  <DIR>         # interactive Flask alignment review
pipeline geo     <DIR>         # geocode entries + build map (needs address fields)
pipeline postprocess <DIR>     # fix + combine + build explorer (post-extraction)
```

Run `pipeline <subcommand> --help` for per-subcommand flags.

The underlying `python main.py <URL> [--flags]` interface still works for advanced use.
Stages always execute in fixed order regardless of flag order. All stages are optional.

## Key stages

| Flag | What it does | Output |
|------|-------------|--------|
| `--download` | Fetch IIIF images | `output/{slug}/` images + `manifest.json` |
| `--surya-ocr` | Surya line-level bbox detection | `*_surya.json` |
| `--gemini-ocr` | Gemini text extraction | `*_{model}.txt` |
| `--align-ocr` | NW alignment of Gemini text to Surya bboxes | `*_aligned.json` |
| `--review-alignment` | Interactive Flask UI to fix bad alignments | updates `*_aligned.json` |
| `--export-alto` | Aligned OCR → ALTO v3 XML (Solr / IIIF Content Search) | `*.alto.xml` |
| `--extract-entries` | NER → structured entries | `entries_{model}.csv` |
| `--geocode` | Geocode entries | `*_geocoded.csv` |
| `--map` | Leaflet HTML map | `entries_*_geocoded.html` |
| `--select-pages` | Browser UI to pick sample pages (once per volume) | `selection.txt` |
| `--generate-prompts` | Gemini-generated OCR + NER prompts (once per collection type) | `ocr_prompt.md`, `ner_prompt.md` |

Full filename contract (who writes/reads every artifact, model auto-detection rules): see "Artifacts and naming conventions" in `docs/pipeline-stages.md`.

## Gemini OCR cost options

`--gemini-ocr` supports two cost-saving modes via the Gemini API:

- **`--flex`** — Flex inference (`service_tier="flex"`): ~50% cheaper than standard pricing, with 1–15 min latency per request. Best for large volumes where real-time throughput isn't needed. Flex is **on by default** everywhere (`pipeline run`/`guided`/`ocr`/`extract` and `python main.py`) — pass `--no-flex` for time-sensitive runs. Only direct invocation of the leaf scripts (`pipeline/run_gemini_ocr.py`, `pipeline/extract_entries.py`) keeps `--flex` opt-in.
- **Batch API** (not yet implemented) — Submit all pages as a single async job; 50% cheaper, up to 24-hour turnaround. Planned for future implementation.

Default model is `gemini-3.1-flash-lite` for both OCR and NER (constants in `utils/models.py`). For higher accuracy use `--ocr-model gemini-2.0-flash`.

## Surya and alignment

- `--surya-ocr` requires GPU or Apple Silicon; slow on CPU
- `--align-ocr` uses Needleman-Wunsch with city/state headings as anchors; runs a second pass on unmatched lines to catch missed columns
- `--review-alignment` is a Flask server — access via `localhost:5000` locally or ngrok/Colab proxy in Colab
- Aligned JSON confidence values: `"line"` (Surya), `"manual"` (user-confirmed via review UI)

## Supported sources

| Source | How to use |
|--------|-----------|
| Library of Congress | Pass any `loc.gov` URL; `--loc-csv` exports metadata |
| Internet Archive | Pass any `archive.org` URL; `--ia-csv` exports metadata |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hadro/directory-pipeline](https://github.com/hadro/directory-pipeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
