---
trigger: always_on
description: Plug and Play for the Mass Desktop AI app (Tauri + React + TypeScript) for local LLM chat, image and video generation via ComfyUI.
---

# Locally Uncensored — Developer Guide

## Project Overview
Plug and Play for the Mass Desktop AI app (Tauri + React + TypeScript) for local LLM chat, image and video generation via ComfyUI.
- **Repo:** PurpleDoubleD/locally-uncensored (35+ stars)
- **Current public version:** v2.3.4 (released 2026-04-20 — Bug #7 persistence fix + Ollama 0.21 compat + Codex loop guard)
- **Branch:** master (all recent work landed)
- v2.3.3 shipped 2026-04-16 as a feature release (Remote Access + Codex streaming + Agent Overhaul + ERNIE + Qwen 3.6). v2.3.4 is a reliability hotfix on top — same feature surface, critical bugs gone.

## Tech Stack
- **Frontend:** React 19, Zustand, Tailwind CSS 4, Framer Motion, Vite 8
- **Backend:** Tauri 2 (Rust), tokio, reqwest
- **Testing:** Vitest 4, pattern `src/**/__tests__/**/*.test.ts`, node environment
- **Build:** `npm run dev` (frontend), `npm run tauri:dev` (full app)

## Key Architecture
```
src/api/comfyui.ts          — Model classification, ComfyUI API, workflow builders, uploadImage()
src/api/dynamic-workflow.ts — Strategy detection + dynamic workflow building (14 strategies)
src/api/comfyui-nodes.ts    — Node discovery + categorization from ComfyUI /object_info
src/api/discover.ts         — Model bundles (14 video, 6 image), CUSTOM_NODE_REGISTRY, downloads
src/api/preflight.ts        — Pre-generation validation (VAE/CLIP/node checks)
src/api/backend.ts          — Tauri IPC abstraction (backendCall, localFetch, comfyuiUrl)
src/api/workflows.ts        — Workflow validation, format conversion, parameter injection
src/stores/downloadStore.ts — Unified download tracking (Zustand) for ComfyUI model downloads
src-tauri/src/commands/      — Rust commands: install, process, download, proxy, etc.
```

## Current Work: v2.3.0 (branch: full-comfyui-fix)

### What's DONE (607 tests passing):
1. **7 new ModelTypes:** mochi, cosmos, cogvideo, svd, framepack, pyramidflow, allegro
2. **7 new WorkflowStrategies** with complete node chains for each model
3. **14 video bundles + 6 image bundles** in discover.ts with HuggingFace URLs
4. **CUSTOM_NODE_REGISTRY** — 5 custom node repos (AnimateDiff, CogVideoX, FramePack, PyramidFlow, Allegro)
5. **install_custom_node** Rust command — git clone + pip install into ComfyUI/custom_nodes/
6. **Onboarding 'comfyui' step** — auto-detect, one-click install, re-scan button, manual path input
7. **Onboarding polish** — window drag region + controls, accent dots (Agent Tutorial style), step indicator dots, tool calling badges, hardware-aware model filtering (VRAM), uncensored/mainstream tabs
8. **Settings ComfyUI section** — status indicator (Running/Stopped/Not Installed), start/stop/restart, install button
9. **Preflight** — all 15 ModelTypes handled (needsUnet check covers all new types)
10. **I2V Image Upload UI** — drag & drop in CreateView for SVD/FramePack, uploadImage() to ComfyUI, filename passed to workflow builders
11. **Unified downloadStore** — Zustand store replaces component-local polling, tracks all ComfyUI downloads globally
12. **DownloadBadge unified** — shows text + image + video downloads, grouped by bundle name with sub-file progress
13. **VRAM tier filter tabs** — All / Lightweight / Mid-Range / High-End for video bundles
14. **installBundleComplete()** — one-click: custom nodes + all model files + ComfyUI restart
15. **isInstalled fix** — exact name match (was: base-name comparison, caused Gemma 4 variant bug)
16. **Default view = chat homepage** (Startseite with LU logo), not Model Manager
17. **6 uncensored video bundles** (Wan 2.1 x2, HunyuanVideo, CogVideoX x2, FramePack)
18. **LTX bug fixed** — workflow was 'wan' instead of 'ltx'
19. **Text model download UX complete** — Ollama pull with streaming progress, HF GGUF with auto-fallback path, both tracked in unified DownloadBadge
20. **isInstalled prefix-match** — Ollama models without tag (hermes3) match installed variants (hermes3:8b)
21. **All 3 download flows Tauri-verified** — Ollama pull (events), HF GGUF (invoke), ComfyUI bundles (invoke) — all arg mappings, command registrations, progress polling confirmed

22. **Tauri .exe download fix (camelCase)** — Rust commands used snake_case params but JS sent camelCase. Downloads silently failed in .exe, worked in dev. Fixed: download_model, download_model_to_path, install_custom_node all use camelCase params now.
23. **Retry button for failed downloads** — per-file retry in DownloadBadge + bundle-level retry in DiscoverModels. Only retries failed files, not completed ones.
24. **Download speed display** — MB/s shown per file and per bundle in DownloadBadge
25. **External links open system browser** — all `target="_blank"` links replaced with `openExternal()` via Tauri shell plugin. Added `shell:allow-open` capability.
26. **Bundle installed detection fixed** — error files no longer count as "complete", bundleStatuses refresh after download, 50% threshold for check_model_sizes (sizeGB values are estimates)
27. **LM Studio not auto-started** — openai provider default changed to `enabled: false`, only activated if detectLocalBackends finds it

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PurpleDoubleD/locally-uncensored](https://github.com/PurpleDoubleD/locally-uncensored) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
