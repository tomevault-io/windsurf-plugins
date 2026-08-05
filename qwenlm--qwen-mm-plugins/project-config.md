---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Qwen-MM-Plugins is an Agent Skills + MCP Tools platform for vision-language models. Each capability lives in one directory under `src/capabilities/<name>/`, holding any of: a `skill/` (the Agent Skill) and a `<import_name>/` MCP-server package — each part optional. Main subsystems:

1. **qwen-mm-plugins-core** — MCP server exposing media reading (images, videos), file visualization, OCR, grounding, segmentation, ASR, and vision chat as MCP tools. `src/capabilities/core/` (skill + `qwen_mm_plugins_core/` server).
2. **qwen-mm-plugins-video-memory** — Hierarchical graph memory for long video QA. 4-level tree: Root → SuperEvent → MacroEvent → Subgraph, with embedding-based semantic search. `src/capabilities/video-memory/` (skill + `qwen_mm_plugins_video_memory/` server).
3. **qwen-mm-plugins-video-edit** — Video-editing skill + image/video/audio **generation** MCP tools (DashScope, via `shared.api_dashscope`). `src/capabilities/video-edit/` (skill + `qwen_mm_plugins_video_edit/` server).
4. **qwen-mm-plugins-blender** — Blender 3D modeling: MCP tools driving a live Blender (execute Python, viewport screenshots, PolyHaven/Sketchfab/Hyper3D/Hunyuan3D assets) + a build→refine→verify skill; needs a running Blender + addon. `src/capabilities/blender/`.
5. **qwen-mm-plugins-freecad** — FreeCAD parametric CAD: MCP tools (create/edit objects, execute Python, named-view screenshots, parts library, CalculiX FEM) + a skill; needs a running FreeCAD + addon. `src/capabilities/freecad/`.
6. **qwen-mm-plugins-edu-agent** — Skill only: turns a math/science problem or image into a step-by-step Chinese explainer video or interactive page. `src/capabilities/edu-agent/`.
7. **qwen-mm-plugins-example** — Template capability (skill + server, 5 demo tools) to copy when adding your own. `src/capabilities/example/`.

## Video Content Questions — MANDATORY Skill Usage

When the user provides a video file path (`.mp4`, `.mkv`, `.avi`, `.mov`, `.webm`) or a directory of multiple videos, and asks about its content:

1. **ALWAYS invoke the `qwen-mm-plugins-video-memory` skill FIRST** — before doing anything else
2. **NEVER use ffmpeg/ffprobe to extract frames directly**
3. **NEVER answer based on a few extracted thumbnails** — a 1-hour video cannot be understood from 6 frames

The qwen-mm-plugins-video-memory skill handles: check existing memory → build if needed → query to answer.

## Common Commands

```bash
# Run MCP server (from source)
python3 src/capabilities/core/qwen_mm_plugins_core

# Install — via each harness's native plugin marketplace (reads .claude-plugin/marketplace.json; codex also .codex-plugin/)
claude plugin marketplace add <repo-url-or-path>
claude plugin install qwen-mm-plugins-core@qwen-mm-plugins

# Test MCP server
printf '{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}\n' | python3 src/capabilities/core/qwen_mm_plugins_core

# Tests / lint
python3 -m pytest tests/
ruff format . && ruff check . --fix
```

## Architecture

```
src/                                         # first-party code (shared library + all capabilities)
├── capabilities/                            # one dir per capability (skill and/or MCP server, each optional)
│   ├── core/                                # vision capability (skill + entry: qwen-mm-plugins-core)
│   │   ├── skill/                           #   Agent Skill (symlink target): SKILL.md + references/
│   │   └── qwen_mm_plugins_core/                  #   MCP server package (dir name == import name)
│   │       ├── __init__.py                  #     __version__ + SPECS (mcp_framework.build_registry) + SYSTEM_DEPS table + list_tools/on_start hooks
│   │       ├── __main__.py                  #     generic shim → mcp_framework.run_main (identical across servers)
│   │       ├── oss.py / stdio_streaming.py   (core-local utils; the shared library is src/shared/)
│   │       ├── readers/                     #     image.py (read_image), video.py (read_video)
│   │       ├── apis/                        #     analysis tools: vision_chat, ocr, grounding (OpenAI-compat via shared.api_openai), segmentation, transcribe_audio, image_search, web_*
│   │       ├── producers/                   #     crop, draw_bbox, save_view
│   │       └── renderers/ visualizers/      #     file rendering + visualize tool
│   ├── video-memory/                        # long-video graph-memory capability
│   │   ├── skill/                           #   SKILL.md + build pipeline (self-contained, flat modules)
│   │   │   └── script/build_memory/         #     build_memory.sh, build_graph.py, pipeline_worker.py, llm_client.py,
│   │   │                                    #     prompts.py, schema.py + embeddings.py (copies of the server's)
│   │   └── qwen_mm_plugins_video_memory/     #   MCP server package
│   │       ├── __init__.py                  #     __version__ + SPECS + SYSTEM_DEPS table + list_tools/on_start hooks
│   │       ├── __main__.py                  #     generic shim → mcp_framework.run_main

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [QwenLM/Qwen-MM-Plugins](https://github.com/QwenLM/Qwen-MM-Plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-04 -->
