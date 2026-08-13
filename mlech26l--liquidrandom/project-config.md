---
trigger: always_on
description: Python package for pseudo-random seed data generation for ML/LLM training diversity.
---

# Liquidrandom

Python package for pseudo-random seed data generation for ML/LLM training diversity.

## Project Structure

```
├── pyproject.toml                  # hatchling build, deps: huggingface-hub, pyarrow
├── .python-version                 # 3.12
├── README.md
├── src/liquidrandom/
│   ├── __init__.py                 # Public API: persona(), job(), image(), etc. + model re-exports
│   ├── py.typed                    # PEP 561 marker
│   ├── _loader.py                  # HuggingFace fetch + Parquet read + in-memory cache (text)
│   ├── _image_loader.py            # Row-group-lazy loader for image categories + tag/chain indices
│   ├── _registry.py                # Category → model class + filename mapping (+ IMAGE_CATEGORIES)
│   └── models/                     # Frozen dataclasses with from_dict() and __str__()
│       ├── __init__.py
│       ├── persona.py … instruction_complexity.py   # 12 text models
│       ├── tool_group.py           # ToolGroup/ToolFunction/ToolVariation
│       └── image_sample.py         # ImageSample (shared by all 11 image categories)
├── tests/
│   ├── test_models.py              # from_dict + __str__ for all text models
│   ├── test_loader.py              # Parquet loading, caching, public API (mocked HF)
│   ├── test_image_models.py        # ImageSample round-trip, to_str, save/to_pil
│   └── test_image_loader.py        # Lazy row-group access, tag filter, chains (mocked HF)
├── preview/                        # Generated sample gallery linked from README (384px JPEGs)
└── seed_generation/                # Separate project with own pyproject.toml
    ├── pyproject.toml              # deps: openai, rich, typer, huggingface-hub, pyarrow, google-genai, pillow
    ├── README.md
    ├── generate.py                 # CLI: generate / generate-tools / generate-images / review-images / upload-only
    ├── config.py                   # Constants and defaults (text)
    ├── categories.py               # 12 CategoryConfig with field specs and prompt templates
    ├── image_config.py             # Image constants (model ids, chain shape, preview settings)
    ├── image_categories.py         # 11 ImageCategoryConfig: taxonomy seeds, tag vocab, edit palettes
    ├── gemini_image.py             # google-genai wrapper: generate/edit/VLM-validate/recompress
    ├── image_sampler.py            # Chain planning (LLM) + Nano Banana generation/edits
    ├── image_viewer.py             # HTML review gallery (quality gate before full runs)
    ├── taxonomy.py                 # Phase 1: BFS taxonomy tree generation (shared by all modalities)
    ├── sampler.py                  # Phase 2: round-robin sample generation (text)
    ├── validator.py                # LLM quality validation per batch (text)
    ├── dedup.py                    # Jaccard similarity dedup on token sets
    ├── llm.py                      # AsyncOpenAI client wrapper with retries
    ├── make_preview_gallery.py     # Build preview/ (small JPEGs + one markdown page) from the parquets
    ├── tag_normalize.py            # Map drifted VLM tags back onto the controlled vocabulary
    ├── uploader.py                 # Consolidate JSONL → Parquet + upload to HF
    └── state.py                    # Checkpoint/resume state
```

## Tooling

- **Package manager**: uv
- **Type checker**: ty (run `uv run ty check src/ tests/`)
- **Tests**: pytest (`uv run pytest tests/`)
- **Typed Python**: all code uses type annotations, `from __future__ import annotations`

## Key Design Decisions

### Data format: Parquet (not JSONL)
Seed data is stored as zstd-compressed Parquet on HuggingFace (`mlech26l/liquidrandom-data`). Parquet gives ~5-10x smaller files than JSONL, and pyarrow is already in the typical ML stack. The intermediate per-leaf files during generation remain JSONL (append-friendly), converted to Parquet at upload time.

### Data loading
- `_loader.py` uses `hf_hub_download()` to fetch a single Parquet file per category (not the whole repo)
- Disk cache handled by huggingface-hub (~/.cache/huggingface/hub/)
- In-memory cache via module-level `_cache` dict avoids re-parsing within a session
- Image categories use `_image_loader.py` instead: files are multi-GB, so it keeps an open `pq.ParquetFile` handle and reads one small row group (written with `row_group_size=64`) per sample. Tag/chain lookups read all columns except `image` into cached posting lists. The eager loaders in `_loader.py` raise `ValueError` for image categories.

### Leaf file naming: SHA-256 hash
Per-leaf sample files use `hashlib.sha256(path)[:16]` as filename to avoid filesystem path length limits from deep taxonomy paths.

### Parallelization
Both taxonomy expansion and sample generation use `asyncio.Semaphore(batch_size)` with `AsyncOpenAI`. Progress updates per-leaf completion (not per-batch) for responsive UI.

### Generation defaults
Tuned so `k` matches `samples_per_leaf` to minimize wasted LLM output:
- `n=22000, k=100, batch_size=32, taxonomy_depth=3, samples_per_leaf=100`
- Yields ~216 leaves, ~100 samples each, ~432 LLM call pairs (generate + validate)

### List field handling in models
All `from_dict()` methods use `list(data["field"] or [])` to handle None values from Parquet columns.

## Seed Data Generation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mlech26l/liquidrandom](https://github.com/mlech26l/liquidrandom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
