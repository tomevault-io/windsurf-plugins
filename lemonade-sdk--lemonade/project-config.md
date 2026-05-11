---
trigger: always_on
description: This file provides guidance to agent driven code reviews when working with this repository.
---

# AGENTS.md

This file provides guidance to agent driven code reviews when working with this repository.

## Project Overview

Lemonade is a local LLM server providing GPU and NPU acceleration for running large language models on consumer hardware. It exposes OpenAI-compatible, Ollama-compatible, and Anthropic-compatible REST APIs, plus a WebSocket Realtime API. It supports multiple backends: llama.cpp, FastFlowLM, RyzenAI, whisper.cpp, stable-diffusion.cpp, and Kokoro TTS.

## Architecture

### Executables

- **lemond** — Pure HTTP server. Handles REST API, routes requests to backends, manages model loading/unloading. Configured via `config.json` in the lemonade cache directory. CLI args: `[cache_dir] [--port PORT] [--host HOST]`.
- **lemonade** — CLI client (`src/cpp/cli/`). Commands: `list`, `pull`, `delete`, `run`, `status`, `logs`, `launch`, `recipes`, `scan`, etc. Communicates with router via HTTP. Discovers running server via UDP beacon.
- **LemonadeServer.exe** (Windows) — SUBSYSTEM:WINDOWS GUI app that embeds `lemond` and shows a system tray icon. Auto-starts via Windows startup folder.
- **lemonade-tray** (macOS/Linux) — Lightweight tray client that connects to a running `lemond`. Platform code in `src/cpp/tray/platform/`.
- **lemonade-server** — Deprecated backwards-compatibility shim. Delegates to `lemond` or `lemonade`.

### Backend Abstraction

`WrappedServer` (`src/cpp/include/lemon/wrapped_server.h`) is the abstract base class. Each backend inherits it and implements `load()`, `unload()`, `chat_completion()`, `completion()`, `responses()`, and optionally `install()` / `download_model()`. Backends run as **subprocesses** — Lemonade forwards HTTP requests to them.

| Backend | Class | Capabilities | Device | Purpose |
|---------|-------|-------------|--------|---------|
| llama.cpp | `LlamaCppServer` | Completion, Embeddings, Reranking | GPU | LLM inference — CPU/GPU (Vulkan, ROCm, Metal) |
| FastFlowLM | `FastFlowLMServer` | Completion, Embeddings, Reranking, Audio | NPU | NPU inference (multi-modal: LLM, ASR, embeddings, reranking) |
| RyzenAI | `RyzenAIServer` | Completion | NPU | Hybrid NPU inference |
| whisper.cpp | `WhisperServer` | Audio | CPU | Audio transcription |
| stable-diffusion.cpp | `SdServer` | Image | CPU | Image generation, editing, variations |
| Kokoro | `KokoroServer` | TTS | CPU | Text-to-speech |

Capability interfaces: `ICompletionServer`, `IEmbeddingsServer`, `IRerankingServer`, `IAudioServer`, `IImageServer`, `ITextToSpeechServer` (defined in `server_capabilities.h`). Use `supports_capability<T>(server)` template for runtime checks.

### Router & Multi-Model Support

`Router` (`src/cpp/server/router.cpp`) manages a vector of `WrappedServer` instances. Routes requests based on model recipe, maintains LRU caches per model type (LLM, embedding, reranking, audio, image, TTS — see `model_types.h`), and enforces NPU exclusivity. Configurable via `--max-loaded-models`. On non-file-not-found errors, the router uses a "nuclear option" — evicts all models and retries the load.

### Model Manager & Recipe System

`ModelManager` (`src/cpp/server/model_manager.cpp`) loads the registry from `src/cpp/resources/server_models.json`. Each model has "recipes" defining which backend and config to use. Backend versions are pinned in `src/cpp/resources/backend_versions.json`. Models download from Hugging Face.

### API Routes

All core endpoints are registered under **4 path prefixes**:
- `/api/v0/` — Legacy
- `/api/v1/` — Current
- `/v0/` — Legacy short
- `/v1/` — OpenAI SDK / LiteLLM compatibility

**Core endpoints:** `chat/completions`, `completions`, `embeddings`, `reranking`, `models`, `models/{id}`, `health`, `pull`, `load`, `unload`, `delete`, `params`, `install`, `uninstall`, `audio/transcriptions`, `audio/speech`, `images/generations`, `images/edits`, `images/variations`, `responses`, `stats`, `system-info`, `system-stats`, `log-level`, `logs/stream`

**Ollama-compatible endpoints** (under `/api/` without version prefix): `chat`, `generate`, `tags`, `show`, `delete`, `pull`, `embed`, `embeddings`, `ps`, `version`

**Anthropic-compatible endpoint:** `POST /api/messages` — supports message completion, tool use, and SSE streaming.

**WebSocket Realtime API**: OpenAI-compatible Realtime protocol for real-time audio transcription. Binds to an OS-assigned port (9000+), exposed via the `websocket_port` field in the `/health` endpoint response.

**Internal endpoints:** `POST /internal/shutdown`

Optional API key auth via `LEMONADE_API_KEY` env var (regular API endpoints) or `LEMONADE_ADMIN_API_KEY` env var (full access including internal endpoints). Clients prefer `LEMONADE_ADMIN_API_KEY` if set. CORS enabled on all routes.

### Desktop & Web App


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lemonade-sdk/lemonade](https://github.com/lemonade-sdk/lemonade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
