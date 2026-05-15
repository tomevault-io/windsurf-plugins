---
trigger: always_on
description: This file orients an AI coding agent (Claude Code, Cursor, Aider, Codex, etc.) to
---

# CLAUDE.md — Agent guide for cull

This file orients an AI coding agent (Claude Code, Cursor, Aider, Codex, etc.) to
the repo. It complements [`README.md`](README.md), which is for human users.

If you're a human reading this: nothing here is required to use the project. Skip to the README.

---

## What cull is

cull is a single-machine curation engine for AI-generated images:

1. **Scrapes** images + their generation prompts from 7+ dedicated sources (Civitai, X/Twitter, Reddit, Discord, ZForFree, generic local folders) plus any URL gallery-dl knows ([scraper_gallery_dl.py](pipeline_code/scraper_gallery_dl.py) — Pixiv, DeviantArt, the booru family, ArtStation, Tumblr, FurAffinity / e621, Imgur, Flickr, …).
2. **Queues** them on the local filesystem under `data/queue/<slug>/<source>/`.
3. **Classifies** each image with a vision-language model (LM Studio or Groq) using a strict JSON schema for structured output. The same call optionally emits a training-ready caption (SD prompt / Booru tags / natural language) — see `CaptionConfig` in [vision_prompt.py](pipeline_code/vision_prompt.py).
4. **Sorts** results into category folders alongside the (possibly auto-generated) `.txt` prompt and a `.vision.json` audit record.
5. **Surfaces** everything through a Flask + Alpine.js admin dashboard
   (`http://localhost:5000`) — pipeline control, scraper toggles, gallery
   browsing, prompt editing, ZIP export, per-source analytics.

The product positioning is *automating taste, not running a model*. The architecture is small and the conventions are load-bearing.

## Conventions you must follow

These are load-bearing — breaking them will silently misroute images.

- **Categories** live in [`pipeline_code/categories.py`](pipeline_code/categories.py). Three tuples: `CATEGORIES` (kept buckets), `TERMINAL_CATEGORIES` (DISCARD/CORRUPT), `ALL_CATEGORIES` (everything). Never inline a category list anywhere else.
- **Vision worker registry** lives in [`pipeline_code/vision_workers.py`](pipeline_code/vision_workers.py). Adding a new provider: register a `WorkerSpec` here AND add it to `dashboard_enhanced.ALLOWED_VISION_WORKERS`. The supervisor maps worker name → script via this registry; mismatched names silently no-op.
- **Vision worker base class:** [`pipeline_code/vision_worker_base.py`](pipeline_code/vision_worker_base.py). Subclass `BaseVisionWorker` and implement `classify_image_bytes`. Don't reimplement the resize / rename / save dance — the base owns it.
- **Queue:** use `queue_manager.save_to_queue` / `get_next_image_round_robin` (or the underlying `Queue` Protocol + `FSQueue`). Do NOT iterate the filesystem yourself; the cache layer in `FSQueue` exists for a reason.
- **Dedup:** every scraper uses `seen_store.SeenStore("name", slug=SLUG)`. Adding a new scraper = one `SeenStore(...)` instance + `seen.add(id)` calls and `seen.flush()` between batches. Don't roll your own JSON file.
- **Credentials:** every scraper uses `credentials.get_required("KEY", scraper="name")` for hard requirements and `get_optional` / `get_keylist` for soft ones. `MissingCredentialError` is a `SystemExit` subclass so the supervisor's cooldown applies on misconfigured scrapers.
- **Logging:** library code uses `pipeline_logging.get_logger(__name__)`. Subprocess workers (scrapers, vision workers) keep their `print(..., flush=True)` calls because the supervisor captures and labels stdout cleanly — see [`pipeline_logging.py`](pipeline_code/pipeline_logging.py) for the reasoning.
- **Paths:** [`pipeline_code/paths.py`](pipeline_code/paths.py) is the single source of truth. Default is `<repo>/data/`. Never hardcode an absolute path.
- **Vision prompt + JSON schema:** [`pipeline_code/vision_prompt.py`](pipeline_code/vision_prompt.py) exposes `build_classification_prompt` + `build_response_format` + `apply_scores`. Every worker MUST send the schema in `response_format` (or its provider equivalent) — empty/invalid JSON failure modes were fixed by structured output, don't regress.
- **Auto-captioning:** the schema's `caption` field is ALWAYS required (strict-mode JSON schemas can't have conditional fields). When `AUTO_CAPTION_ENABLED=false`, the prompt instructs the model to return an empty string. When `true`, the prompt swaps in style-specific instructions (`sd_prompt` / `booru_tags` / `natural_language`) and `vision_worker_base._finalise` writes the caption to `<image>.txt`. Existing source-side prompts are preserved unless `AUTO_CAPTION_OVERWRITE=true`.
- **Prompt-required gate:** the `REQUIRE_PROMPT` env var (default `true`) governs whether scrapers reject images that have no prompt / a too-short prompt. The single source of truth for the gate logic is [`topic_filter.prompt_optional()`](pipeline_code/topic_filter.py); every scraper checks it before applying its own length floor. If you add a scraper, follow the pattern.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tlennon-ie/cull](https://github.com/tlennon-ie/cull) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
