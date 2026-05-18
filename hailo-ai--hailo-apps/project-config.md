---
trigger: always_on
description: > This repository is designed for **agentic-first development**. AI coding agents can build complete, production-ready Hailo AI applications by following the structured instructions, skills, and prompts in this `.github/` directory — without manually writing code.
---

# Hailo Apps — Copilot Global Instructions

> This repository is designed for **agentic-first development**. AI coding agents can build complete, production-ready Hailo AI applications by following the structured instructions, skills, and prompts in this `.github/` directory — without manually writing code.

## Repository Identity

- **Name**: hailo-apps
- **Purpose**: Production-grade AI vision & generative-AI applications running on Hailo accelerators (Hailo-8, Hailo-8L, Hailo-10H)
- **Stack**: Python 3.10+, GStreamer, HailoRT, TAPPAS, OpenCV, hailo_platform SDK

## Architecture at a Glance

| Layer | Description |
|---|---|
| **Core Framework** (`hailo_apps/python/core/`) | GStreamerApp base class, pipeline helpers, parsers, logging, HEF utilities |
| **Pipeline Apps** (`hailo_apps/python/pipeline_apps/`) | GStreamer-based video pipelines (detection, pose, segmentation, etc.) |
| **Standalone Apps** (`hailo_apps/python/standalone_apps/`) | Direct inference apps using HailoInfer + OpenCV (no GStreamer) |
| **Gen AI Apps** (`hailo_apps/python/gen_ai_apps/`) | Hailo-10H generative AI: VLM, LLM, Whisper, Voice Assistant, Agent |
| **Postprocess** (`hailo_apps/postprocess/`) | C++ shared libraries for model-specific postprocessing |
| **Config** (`hailo_apps/config/`) | YAML-driven model registry, resource paths, test definitions |

## Critical Conventions (MUST FOLLOW)

1. **Imports are always absolute**: `from hailo_apps.python.core.common.xyz import ...`
2. **HEF resolution**: Always use `resolve_hef_path(path, app_name, arch)` — never hardcode paths
3. **Device sharing**: Always use `SHARED_VDEVICE_GROUP_ID` when creating `VDevice`
4. **Logging**: Use `get_logger(__name__)` from `hailo_apps.python.core.common.hailo_logger`
5. **CLI parsers**: Use `get_pipeline_parser()` for GStreamer apps, `get_standalone_parser()` for standalone/gen-ai apps
6. **Architecture detection**: Use `detect_hailo_arch()` or `--arch` flag; never assume hardware
7. **Entry points**: App must have a `main()` or `if __name__ == "__main__"` block

## Dynamic Context Loading

> **Do NOT read all files.** Use the routing table below to load **only** the files relevant to the current task. This saves tokens and keeps context focused.

### Context Routing Table

Based on what the task involves, read **only** the matching rows:

| If the task mentions... | Read these files |
|---|---|
| **VLM, vision, image understanding** | `skills/hl-build-vlm-app/SKILL.md`, `toolsets/vlm-backend-api.md`, `memory/gen_ai_patterns.md` |
| **LLM, chat, text generation** | `skills/hl-build-llm-app/SKILL.md`, `instructions/gen-ai-development.md`, `toolsets/gen-ai-utilities.md`, `memory/gen_ai_patterns.md` |
| **Agent, tools, function calling** | `skills/hl-build-agent-app/SKILL.md`, `toolsets/gen-ai-utilities.md`, `memory/gen_ai_patterns.md` |
| **Voice, STT, TTS, Whisper, speech** | `skills/hl-build-voice-app/SKILL.md`, `toolsets/gen-ai-utilities.md` |
| **Pipeline, GStreamer, video, stream** | `skills/hl-build-pipeline-app/SKILL.md`, `instructions/gstreamer-pipelines.md`, `toolsets/gstreamer-elements.md`, `memory/pipeline_optimization.md` — **Fast path for simple variants** (detection filter, counter, custom overlay): SKILL.md + `memory/common_pitfalls.md` is sufficient |
| **Game, interactive, pose game** | `skills/hl-build-pipeline-app/SKILL.md`, `toolsets/pose-keypoints.md`, `toolsets/core-framework-api.md`, `memory/common_pitfalls.md` |
| **Standalone, OpenCV, HailoInfer** | `skills/hl-build-standalone-app/SKILL.md`, `toolsets/core-framework-api.md` |
| **Camera, USB, RPi, capture** | `skills/hl-camera.md`, `memory/camera_and_display.md` |
| **HEF, model, download, config** | `skills/hl-model-management.md`, `toolsets/hailort-api.md`, `memory/hailo_platform_api.md` |
| **Monitoring, events, alerts** | `skills/hl-monitoring.md`, `skills/hl-event-detection.md` |
| **Testing, validation, pytest** | `skills/hl-validate.md`, `instructions/testing-patterns.md` |
| **Complex multi-file app** | `instructions/orchestration.md`, `skills/hl-plan-and-execute.md`, `instructions/agent-protocols.md` |
| **Building any new app** | The specialist agent (loaded via VS Code `@agent`) handles interactive flow. If not using agents, read `instructions/orchestration.md` and the relevant `skills/hl-build-*/SKILL.md` |
| **ALWAYS read (every task)** | `memory/common_pitfalls.md`, `instructions/coding-standards.md` |

All paths above are relative to `.github/`. The knowledge base at `.github/knowledge/knowledge_base.yaml` can be checked when you need recipes or patterns.

### Persistent Memory

```
.github/memory/
├── MEMORY.md                  ← Index — read this first
├── gen_ai_patterns.md         ← VLM/LLM architecture, multiprocessing, gotchas
├── pipeline_optimization.md   ← GStreamer bottlenecks, queue tuning, scheduler fixes
├── camera_and_display.md      ← Camera init, BGR/RGB, OpenCV patterns
├── hailo_platform_api.md      ← VDevice, VLM.generate(), HEF resolution
└── common_pitfalls.md         ← Bugs found, anti-patterns to avoid
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hailo-ai/hailo-apps](https://github.com/hailo-ai/hailo-apps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
