---
trigger: always_on
description: SPDX-FileCopyrightText: Copyright (c) 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
---

<!--
  SPDX-FileCopyrightText: Copyright (c) 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
  SPDX-License-Identifier: Apache-2.0
-->

# xr-ai — Working Conventions

The contract every change must satisfy. Topic deep-dives live in `docs/`;
historical decisions in `docs/changelog.md`.

## Architecture (sketch)

```
client-samples/     # Platform clients (Android, iOS/visionOS, Web)
server-runtime/     # XR-Media-Hub core + LiveKit transport
agent-sdk/          # Three packages:
                    #   xr-ai-agent   — IPC client library (pyzmq + msgpack only)
                    #   xr-ai-models  — LLM/VLM/STT/TTS service protocols + OpenAI-compat clients
                    #   xr-ai-pipecat — optional Pipecat transport bridge (heavier deps)
utils/              # Shared infra: launcher, logging, vad, vllm, voicegate
cloudxr-runtime/    # Shared CloudXR OpenXR runtime + WSS proxy (opt-in)
ai-services/        # OpenAI-compatible inference servers (VLM, STT, TTS, LLM)
agent-mcp-servers/  # MCP adapters: oxr, render, transcript, vec, video, vlm
agent-samples/      # End-to-end agent demos
tests/              # Multi-client / multi-agent integration tests
docs/               # Topic deep-dives + changelog
models/             # Gitignored model-weight cache (per-YAML model_cache target)
deps/               # Gitignored downloaded binaries (e.g. LOVR AppImage)
```

## Hard rules

- **One hub, many clients, many agents.** Hub fans inbound to every
  `ProcessorEndpoint`; return traffic goes only to the originating client.
- **Agents talk to the hub via IPC only.** LiveKit is an internal transport
  detail — never surface it to agents.
- **`agent-sdk/xr-ai-agent` depends only on `pyzmq` + `msgpack`.** No
  LiveKit, FastAPI, or uvicorn. `agent-sdk/xr-ai-pipecat` is a separate
  optional package with heavier deps (pipecat-ai, scipy, numpy, httpx,
  fastmcp); it bridges `ProcessorEndpoint` to Pipecat pipelines.
- **All HTTP calls to AI services go through `agent-sdk/xr-ai-models`.**
  Workers and MCP servers depend on its four protocols
  (`LLMService`, `VLMService`, `STTService`, `TTSService`) and construct
  clients from a per-sample `yaml/models.yaml` via `make_llm` /
  `make_vlm` / `make_stt` / `make_tts`.  Hand-rolled `httpx` clients
  against `/v1/chat/completions`, `/v1/audio/transcriptions`, or
  `/v1/audio/speech` are forbidden — model quirks belong in this one
  package's presets, not in callers. No vendor SDKs (no `openai`, no
  `anthropic`, no `litellm`); all in-tree backends speak
  OpenAI-compatible HTTP.
- **Workers never import from `server-runtime` or `xr_ai_launcher`.** Only
  `xr_ai_agent` + `xr_ai_models` + task-specific libs (numpy, torch, …).
- **MCP servers are the agent's only interface to XR data and rendering.**
- **No API keys or tokens in source files** — use env vars or
  `xr_media_hub.yaml`. See `docs/credentials.md`.

## Process model essentials

Each sample has two sub-projects:

| Sub-project | Role | Dependencies |
|---|---|---|
| `<sample>/` | Orchestrator — declares `PROCESSES`, calls `run_stack` | `xr-ai-launcher` only |
| `<sample>/worker/` | Agent worker — connects to hub via IPC | `xr-ai-agent` + task libs |

- Processes start serially in declaration order; each must `Path(--ready-file).touch()`
  when ready.
- `xr_media_hub` always runs as its own process — never embedded.
- `run_stack` is fail-fast: any process exit terminates the stack.
- Process management lives in `utils/xr-ai-launcher/`, not inside any process it manages.

Full mechanics and the `Process(...)` declaration form: `docs/process-model.md`.

## Adding a sample

Pick a kebab-case name (e.g. `simple-vlm-example`); derive everything else
mechanically:

| Thing | Convention | Example |
|---|---|---|
| Sample directory | `agent-samples/<kebab>/` | `simple-vlm-example/` |
| Orchestrator entry | `<snake_name>` | `simple_vlm_example` |
| Worker entry | `<snake_name>_worker` | `simple_vlm_example_worker` |
| Agent class | `<CamelName>Agent` | `SimpleVlmAgent` |

**Worker code rules** (apply to every sample worker):

- Only import from `xr_ai_agent` for IPC types.
- `_HUB_PUB` / `_HUB_PUSH` are module-level constants, not magic strings.
- Wire `SIGINT` and `SIGTERM` to `agent.shutdown()`; wrap `await agent.run()`
  in `try/finally` calling `shutdown()`.
- `shutdown()` is synchronous (signal-handler safe). Cancel asyncio tasks
  first, then `ep.stop()` + `ep.close()`.
- Callbacks are `async def` even if the work inside is sync.
- CPU-bound work goes through `loop.run_in_executor(...)` — never block the
  event loop.
- Imports are absolute (flat module layout). No `__init__.py` or `__main__.py`.

**Checklist for a new sample:**

- [ ] `agent-samples/<name>/pyproject.toml` — orchestrator, deps: `xr-ai-launcher` only
- [ ] `agent-samples/<name>/worker/pyproject.toml` — worker, deps: `xr-ai-agent` + task libs (list every `.py` in `only-include`)
- [ ] `agent-samples/<name>/main.py` — exact orchestrator boilerplate
- [ ] `agent-samples/<name>/worker/<snake_name>_worker.py` — entry point + (optional) split helpers
- [ ] `agent-samples/<name>/yaml/xr_media_hub.yaml` — hub config
- [ ] `agent-samples/<name>/yaml/<command>.yaml` — one per process that needs config

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA/xr-ai](https://github.com/NVIDIA/xr-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
