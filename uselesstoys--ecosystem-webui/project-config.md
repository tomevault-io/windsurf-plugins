---
trigger: always_on
description: This is the primary reference for any AI agent working in this repository.
---

# AGENTS.md — Project Guide for AI Coding Agents

This is the primary reference for any AI agent working in this repository.
Agent-specific files (`CLAUDE.md`, `GEMINI.md`, `QWEN.md`) point here and add
per-agent notes on top.

---

## CRITICAL RULES — Read These First

### This is a WEB UI Project (NOT Jupyter Notebooks)

This project uses **FastAPI + Next.js** for the interface, not Jupyter notebooks.

- **NO** `.ipynb` files, notebook cells, or ipywidgets — ever
- **YES** FastAPI backend (Python) + Next.js frontend (React/TypeScript)
- Access via web browser at `http://localhost:3000`
- If you see references to "notebooks" or "Jupyter" in existing code, they are outdated

### Vendored Backend — No Submodules

`trainer/derrian_backend/` is committed directly into the repository.

- **DO** treat it as regular source code
- **DO NOT** run `git submodule` commands — there are no submodules
- **DO NOT** add git submodules to this repository
- **TRY NOT TO** modify vendored backend files casually, but editing them to fix or improve something we need is expected — we carry a live patch-set on this tree (see ``upstream_sync_methodology``). Preserve existing patches when you touch it.

### Do Not Change Server Bindings

Do not change `0.0.0.0` to `localhost` or `127.0.0.1`:

```python
uvicorn.run(app, host="0.0.0.0", port=8000)  # correct — leave as-is
```

This application deploys to VastAI and RunPod cloud GPU instances. `0.0.0.0`
is required for external access. Security is handled by the platform firewall
and Cloudflare proxy. Do not change this even if a security scanner flags it.

### Do Not Modify Git Remotes

Never run commands that change remote configuration:

```bash
# Never run:
git remote set-url origin ...
git remote add ...
git remote remove ...
```

### Do Not Create Audit Files

Do not create markdown audit files (`API_AUDIT.md`, `ENDPOINT_AUDIT.md`, etc.)
unless the user specifically asks for one. Report findings directly in the
conversation instead.

---

## Repository Overview

A **web-based** LoRA training environment for training LoRA models for Stable
Diffusion image generation. Uses a FastAPI backend and Next.js frontend,
designed to run cross-platform.

**Architecture**: FastAPI (Python) + Next.js 15 (React/TypeScript) + Kohya SS

**Deployment targets** (all equally supported):
- **Windows local** — primary development environment; code must work here first
- **VastAI** — Linux cloud GPU instances with auto-provisioning and supervisor
- **RunPod** — Linux cloud GPU instances via `provision_runpod.sh`
- **Training requirement** — NVIDIA GPU with CUDA 12.1+

**Not supported**: macOS. Do not write Mac-specific code or instructions.

---

## Key Components

### Backend — `api/`

FastAPI route handlers:

| File | Purpose |
|------|---------|
| `api/main.py` | FastAPI app entry point |
| `api/routes/dataset.py` | Dataset upload and management |
| `api/routes/training.py` | Training configuration and execution |
| `api/routes/models.py` | Model downloads (HuggingFace / Civitai) |
| `api/routes/utilities.py` | LoRA utilities, HuggingFace uploads, Chattiori merge/bake |
| `api/routes/config.py` | Configuration management |
| `api/routes/files.py` | File operations and browsing |
| `api/routes/settings.py` | Application settings |
| `api/routes/civitai.py` | Civitai integration |
| `api/routes/debug.py` | Debug and diagnostics |

**Deprecated routes** (code moved to `*deprecated/` but endpoints still work):
| `POST /checkpoint/merge-weighted` | `services/deprecated/block_weight_merge.py` |
| `POST /lora/merge-to-checkpoint` (Anima bake) | `custom/deprecated/anima_merge_lora.py` |

### Service Layer — `services/`

Business logic. Route handlers call services; services call the training
backend. Never call Kohya scripts directly from routes.

| File / Dir | Purpose |
|-----------|---------|
| `training_service.py` | Training job orchestration |
| `tagging_service.py` | WD14 image tagging |
| `captioning_service.py` | BLIP / GIT captioning |
| `dataset_service.py` | Dataset processing and validation |
| `caption_service.py` | Caption file management |
| `model_service.py` | Model download and management |
| `lora_service.py` | LoRA utilities |
| `chattiori_service.py` | Chattiori merge/bake subprocess wrapper |
| `websocket.py` | WebSocket handlers for real-time logs |
| `jobs/` | Job management system |
| `trainers/` | Training backend integration (Kohya SS) |
| `models/` | Pydantic data models and schemas |
| `core/` | Core utilities and path validation |
| `deprecated/` | Deprecated modules (block_weight_merge, anima_merge) — kept for reference |

### Frontend — `frontend/`

Next.js 15 with React 19, TypeScript, and Tailwind CSS v4.

| Dir | Purpose |
|-----|---------|
| `frontend/app/` | Next.js App Router pages |
| `frontend/components/` | React components |
| `frontend/lib/` | API client (`api.ts`) and utilities |
| `frontend/hooks/` | Custom React hooks |

All backend communication goes through `frontend/lib/api.ts`. Do not fetch the
backend directly from components.

### Training Backend — `trainer/derrian_backend/`

Vendored Kohya SS distribution (committed directly, not a submodule):

| Dir | Purpose |
|-----|---------|
| `sd_scripts/` | Kohya SS training scripts |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [UselessToys/Ecosystem_WebUI](https://github.com/UselessToys/Ecosystem_WebUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-19 -->
