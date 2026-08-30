---
trigger: always_on
description: Interactive test console for assessing Vision-Language Models (VLMs) on object detection. Uses an iterative **Detector-Judge pipeline** (LangGraph) where a detector proposes bounding boxes, a judge critiques them, and the loop repeats with structured feedback. Gradio console provides **Batch Processing** (top Live Results Explorer + bottom Input/Configuration columns), **Draw & Recognize** (custom canvas + DetectionViewer), **Real-Time Detection** (webcam/video with tracker + same-window overlay
---

# AGENTS.md — LLM Object Detection Testing Console

## Project Overview
Interactive test console for assessing Vision-Language Models (VLMs) on object detection. Uses an iterative **Detector-Judge pipeline** (LangGraph) where a detector proposes bounding boxes, a judge critiques them, and the loop repeats with structured feedback. Gradio console provides **Batch Processing** (top Live Results Explorer + bottom Input/Configuration columns), **Draw & Recognize** (custom canvas + DetectionViewer), **Real-Time Detection** (webcam/video with tracker + same-window overlay) and **Real-Time Draw** (live capture + draw) tabs. All tabs share a unified **DetectionViewer** (`llmog/detection_viewer`) for client-side box rendering (WebP cache, no server PIL re-encode).

## Package Management
- **Tool**: `uv` (fast Python installer/resolver)
- **Python**: 3.12+ (see `.python-version`)
- **Install**: `uv sync` (after `scripts/install_llama_cpp.sh` for llama.cpp) – `pyproject.toml` now includes `detection_viewer` in `tool.setuptools.packages.find`

## Entry Points
| Command | Module | Description |
|---------|--------|-------------|
| `uv run llmog` | `main:main` | Unified CLI; dispatches by `--task` (`free_detection` / `auto_label`) |
| `uv run detection-cli` | `free_detection:main` | Shortcut for `llmog --task free_detection` (detector/judge loop on `--image` paths) |
| `uv run auto-annotation` | `auto_annotation:main` | Shortcut for `llmog --task auto_label` (batch YOLO relabeling from a `data.yaml`) |
| `uv run detection-gui` | `interface.gui:main` | Launch Gradio console (`app_builder.build_app()`) |

Single source of truth for CLI flags: `llmog/schemes/argument.py:PipelineConfig` (pydantic v2). `llmog/main.py:build_parser` mirrors every field onto `argparse`; `parse_args()` overlays optional `--config <yaml>` and constructs validated `PipelineConfig`.

## Key Directories
- `llmog/` — Package root (`tool.setuptools.package-dir = {"": "llmog"}`)
- `llmog/schemes/` — `PipelineConfig` + argparse mirror
- `llmog/main.py` — Unified CLI dispatcher (`--task free_detection | auto_label`)
- `llmog/free_detection/` — Detector/Judge pipeline package
- `llmog/free_detection/agent/` — LangGraph nodes (`preprocess`, `detector`, `crop_verify`, `judge`, `loop`, `finalize`), `pipeline.py`, `state.py`, `visuals.py`, `client_utils.py` (429-aware retry)
- `llmog/detection_viewer/` — New `DetectionViewer` (gr.HTML) – `__init__.py`, `static/template.html|style.css|script.js`, `py.typed` – client-side canvas, WebP cache with dedup (`_WEBP_URL_CACHE`)
- `llmog/auto_annotation/` — Batch YOLO relabeling
- `llmog/prompts/` — Markdown templates for detector/judge/realtime (`detector_agent.md`, `realtime_detector.md`, `auto_label_classifier.md`) loaded via DynaPrompt
- `llmog/servers/` — `LlamaServerManager`/`VllmServerManager` + `servers_factory`
- `llmog/interface/` — Gradio console
  - `app_builder.py` — Aggregator, builds 6 tabs, wires global endpoint
  - `viewer_utils.py` — Shared adapters (`pipeline_detections_to_annotations`, `region_results_to_annotations`, `realtime_boxes_to_annotations`, `build_prep_config`, `build_viewer_payload`) + palette/WebP helpers
  - `tab_server.py` — Unified **Model / Endpoint** tab (Local vs External API global state)
  - `tab_draw.py` — Draw & Recognize (custom HTML5 canvas `CustomCanvasController`, `gr.UploadButton` upload, `DetectionViewer` results)
  - `tab_realtime_interactive.py` — **Real-Time Draw** (live capture `CustomCanvasControllerRT` with RT-specific ids, `gr.Image` webcam → canvas)
  - `batch/` — `components.py` (Batch UI), `runner.py` (threaded batch, lazy grid, 1600px cache cap), `explorer.py` (lazy grid + viewer payload), `reclassification.py` (crop classify: sequential / `asyncio.gather` parallel / batched single-request)
  - `realtime/` — `ui.py` (stream + video + same-window overlay), `handlers.py` (motion-gate diff 64×64, frame hash dedup, `build_prep_config`), `utils.py` (downscale to 1280, JPEG q85), `state.py` (SessionDetector, `resolve_endpoint`, pipeline cache)
  - `console.css` / `console.js` / `console_theme.py` — Dark terminal theme, DetectionViewer dark overrides, draw-tab responsive
- `scripts/` — Linux install scripts

## Core Modules
- `free_detection/agent/pipeline.py` — `ObjectDetectionPipeline` (LangGraph `compile()`, `run()`, `run_inference()`, `judge_detections()`)
- `free_detection/agent/graph.py` — `build_detection_graph()`
- `free_detection/agent/client_utils.py` — `_call_with_retries()` (429-aware, parses `RetryInfo/retryDelay`, exponential backoff + jitter, global Gemini 4s interval, 5 retries)
- `free_detection/agent/nodes/detector.py` — `_run_tiled_detection()` caps `max_workers=1` for Gemini to respect 15 RPM
- `free_detection/agent/visuals.py` — `render_detections()`, `draw_grid()`, `pil_to_data_uri()`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mohamed-em2m/llmog](https://github.com/mohamed-em2m/llmog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
