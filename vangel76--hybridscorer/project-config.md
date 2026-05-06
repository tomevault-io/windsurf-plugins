---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Syntax check (fast, no GPU needed)
python -m py_compile Hybrid-Scorer.py

# Run the app
./run-Hybrid-Scorer.sh        # Linux
run-Hybrid-Scorer-windows.bat # Windows

# Setup / update environment
./setup_update-linux.sh       # Linux
setup_update-windows.bat      # Windows
```

Default app URL: `http://localhost:7862`

## Architecture

**Modular Gradio app.** `Hybrid-Scorer.py` (~838 lines) is pure Gradio wiring; all logic lives in `lib/`.

```
lib/
  config.py          # constants, method labels, model IDs
  utils.py           # pure utility functions
  backend.py         # ModelBackend class
  scoring.py         # score_all, encode_all_promptmatch_images
  helpers.py         # UI text helpers, score helpers, prompt helpers
  state.py           # get_state_defaults(), init_state()
  state_helpers.py   # state-management helpers (accept state param)
  loaders.py         # ensure_*_model, ensure_*_backend, feature caches
  view.py            # gallery, histogram, current_view, render_view_with_controls
  callbacks/
    scoring.py       # score_folder, find_similar_images, find_same_person_images
    prompts.py       # generate_prompt_from_preview, run_*_prompt_variant
    ui.py            # handle_thumb_action, export_files, threshold callbacks, etc.
static/
  style.css          # all app CSS
  app.js             # all app JS (tooltip dict injected at runtime)
```

`create_app()` is the center of gravity:
- owns the shared mutable `state` dict
- binds extracted callbacks with `functools.partial(func, state)` or `partial(func, state, device)`
- builds the UI and injects JS/CSS
- creates initial backend/model objects

**Note:** `.select()` event handlers that use `gr.SelectData` must be defined as thin wrapper closures in `create_app()` rather than bound with `partial` — Gradio cannot see the `SelectData` annotation through a partial.

UI behavior is callback-driven. Some behavior lives in injected JS, not Python. Always inspect both before changing UI logic.

### Six scoring modes

| Mode | Backend | Cache used |
|---|---|---|
| **PromptMatch** | CLIP-family (open_clip, SigLIP, etc.) | per-folder image embeddings |
| **ImageReward** | ImageReward-v1.0 | per-folder base + penalty scores |
| **Similarity** | reuses PromptMatch embeddings | same as PromptMatch |
| **SamePerson** | InsightFace buffalo_l (ONNX) | per-folder face embeddings |
| **LLM Search** | PromptMatch shortlist → vision-language rerank | PromptMatch embeddings + LLM captions |
| **ObjectSearch** | DINOv2 ViT-B/14 patch features + FAISS (CPU) / GPU matmul | per-folder patch embeddings (256 patches/image, in-memory) |

### LLM Search flow
1. PromptMatch builds a shortlist from the text query
2. A local vision-language backend reranks the shortlist (Florence-2, JoyCaption HF, or JoyCaption GGUF)
3. Non-shortlisted images get a deterministic reject-floor score
4. HF JoyCaption uses batched inference (`score_candidates_batch`, `LLMSEARCH_JOYCAPTION_HF_BATCH_SIZE=4`); GGUF is sequential

### ObjectSearch flow
1. User sets a query image (upload, paste, or gallery preview) in accordion 3
2. `ensure_objectsearch_feature_cache` extracts DINOv2 patch tokens for the whole folder, builds a `faiss.IndexFlatIP` CPU index and (if CUDA available) a GPU tensor copy of all patches
3. `encode_single_objectsearch_query` extracts patch tokens from the query image
4. `score_objectsearch_cached_features` runs GPU `torch.mm` (or FAISS CPU fallback), aggregates best-match-per-query-patch per gallery image, returns mean score
5. Scores feed into the standard threshold/split/export flow — uses `uses_similarity_topn` (top-N slider) same as Similarity and SamePerson
- State keys: `os_cached_*`, `dinov2_backend`, `objectsearch_query_fname/source`
- `release_inactive_gpu_models` clears `dinov2_backend`, `os_cached_faiss_index`, and `os_cached_patch_gpu_tensor`

### VRAM management
`release_inactive_gpu_models(target_method)` is called at the top of `score_folder`, `find_similar_images`, `find_same_person_images`, and `find_objectsearch_images` — before any model loads. It frees models not needed for the incoming method. The PromptMatch CLIP `state["backend"]` is never released because it is always needed for shortlist embeddings.

### Cache and proxy system
- Cache mode defaults: Windows → project-local (`models/`, `cache/`); Linux → system (`~/.cache/...`)
- Override: `HYBRIDSCORER_CACHE_MODE=project` or `HYBRIDSCORER_CACHE_MODE=system`
- On Linux (system mode), proxy thumbnails go to `/dev/shm` (RAM-backed tmpfs) with fallback to `tempfile.gettempdir()`
- `get_cache_config()` is `@lru_cache(maxsize=1)` — runs once per session

### Sidebar layout
The sidebar has four collapsible accordion sections (mutually exclusive — opening one closes all others; only **1. Setup** is open on first load):
- `#hy-acc-setup` — 1. Setup
- `#hy-acc-scoring` — 2. Scoring & Method/Settings
- `#hy-acc-search-image` — 3. Search + Prompt from image
- `#hy-acc-export` — 4. Export


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vangel76/HybridScorer](https://github.com/vangel76/HybridScorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
