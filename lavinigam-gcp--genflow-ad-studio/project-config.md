---
trigger: always_on
description: AI-powered 30-second video commercial generator. Product image in, finished ad out.
---

# Genflow Ad Studio -- Development Guide

AI-powered 30-second video commercial generator. Product image in, finished ad out.
Stack: FastAPI + React 19 + MUI v7 | Gemini 3 Pro/Flash/Image + Imagen 4 + Veo 3.1 | FFmpeg

## Commands

```bash
make setup           # Full first-time setup (install + GCS + sample images)
make dev             # Run backend (8000) + frontend (3000)
make check           # Type-check backend imports + frontend TSC + validate assets
make test            # Full system test (starts servers, tests API + frontend + auth + assets)
make reset-db        # Delete SQLite DB + legacy job files (fixes schema errors)
make generate-samples # Generate missing sample product images via AI
make help            # Show all available commands
```

**Always run `make check` before finishing any task.**

## Codebase Structure

```
backend/
  main.py                     # FastAPI app + route registration + /asset static mount
  scripts/                    # Utility scripts (generate_samples.py)
  app/
    dependencies.py           # DI container (@lru_cache singletons)
    config.py                 # Settings via pydantic-settings + .env
    ai/    {gemini, gemini_image, imagen, veo, retry, prompts}.py
    models/ {job, script, avatar, storyboard, video, review, sse, common}.py
    services/ {pipeline, script, avatar, storyboard, video, stitch, qc, review, bulk, input}_service.py
    api/    {pipeline, jobs, bulk, review, assets, health, config_api, input}.py
    jobs/   {store, runner, events}.py
    utils/  {ffmpeg, csv_parser, json_parser, sse_log_handler}.py
    storage/ {local, gcs}.py
  output/
    samples/                  # 9 AI-generated product images (checked into git)
frontend/
  public/                     # Static assets (logo, favicons, web manifest)
  src/
    api/     {client, pipeline}.ts
    types/   index.ts            # Must mirror backend Pydantic models exactly
    constants/ controls.ts       # Shared UI constants (models, tones, defaults)
    store/   {pipeline, review, bulk}Store.ts
    components/ {pipeline/, review/, common/, layout/, pages/}
asset/                          # Generated architecture diagrams (served at /asset/)
.docs/diagram-generator/        # Diagram generation CLI + JSON prompts
```

## Architecture Rules

- URL paths via `storage.to_url_path()` -- never return filesystem paths to the frontend
- RPC-style POST routes with request body -- not RESTful resource URLs
- New services must register in `backend/app/dependencies.py` (`@lru_cache` singleton)
- `@async_retry` on all AI SDK calls (Gemini + Veo)
- Frontend types (`types/index.ts`) must mirror backend Pydantic models field-for-field
- Veo outputs VFR video -- always preprocess to 24fps CFR before stitching
- QC feedback loop: generate, QC score, rewrite prompt, regenerate (max 3 attempts)
- Manual regen auto-carries `previous_qc_report` for QC-informed prompt rewriting
- `image_url` accepts local `/output/...` paths -- services detect prefix and read from disk
- Video duration = user-selectable 4/6/8s (8s auto-enforced with reference images or resolution >= 1080p)
- `generate_audio` toggle: configurable via VideoPlayer Switch (default True)
- File uploads: use `api.upload()` with FormData -- `api.post()` is for JSON only
- `detailed_avatar_description` must be identical across all scenes for Veo consistency
- Same Veo `seed` across all scenes for character/voice consistency
- Veo API: `image` (first-frame) and `reference_images` (asset refs) are mutually exclusive
- Scene-to-scene continuity: last frame extracted and passed as asset reference to next scene
- Imagen 4 does NOT support `negative_prompt` -- use positive prompting only
- Transition types in script map to FFmpeg xfade effects via `TRANSITION_MAP` in `ffmpeg.py`
- Every prompt template field must be wired end-to-end: model field, service, template `.format()`
- Persisted Pydantic models must use optional fields with defaults for backward compatibility
- `JobStore.list_jobs()` catches and skips corrupted rows -- one bad job must not crash the app
- `HowItWorksPage.tsx` step data uses `subtitle` + `bullets[]` (demo talking points) -- not paragraph prose
- Architecture diagrams must be demo-friendly: no code references, no jargon -- plain language only
- All diagrams match the visual style of `pipeline-flow.webp`: warm gray background, white cards, Google colors

## Adding a Feature

1. Define Pydantic model in `backend/app/models/` (snake_case fields, string enums)
2. Create service in `backend/app/services/` (constructor injection, async methods)
3. Register in `backend/app/dependencies.py` -- add `get_foo_service()` with `@lru_cache`
4. Add POST route in `backend/app/api/` -- request body, no path params
5. Register router in `backend/main.py` with `/api/v1/` prefix
6. Add matching TypeScript interface in `frontend/src/types/index.ts`
7. Add API function in `frontend/src/api/pipeline.ts`
8. Update Zustand store with new state fields
9. Add shared constants to `frontend/src/constants/controls.ts`
10. Create/update component in `frontend/src/components/pipeline/`
11. Add SSE event type if step emits progress

## Code Style

### Python (Backend)

- Async-first with FastAPI
- Pydantic models with snake_case fields

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lavinigam-gcp/genflow-ad-studio](https://github.com/lavinigam-gcp/genflow-ad-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
