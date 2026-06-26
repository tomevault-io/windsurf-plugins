---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Ghost Creator AI is a Windows desktop app that generates documentary-style short/long videos from a topic (research → script → critic → review → voice → footage → assembly → YouTube upload). It ships as an **Electron + React** GUI shell driving a **Python FastAPI sidecar**, with the pipeline itself implemented as a **stateful LangGraph** graph checkpointed to SQLite.

## Architecture: two processes, one machine

The Electron main process (`electron/`) spawns the Python API as a child process and talks to it over HTTP/WebSocket on `127.0.0.1:8766`.

- **`electron/python-bridge.ts`** resolves which Python to run, in order: `venv/Scripts/python.exe -m api.server` (dev) → `resources/GhostCreatorAPI/GhostCreatorAPI.exe` (packaged onedir) → `python -m api.server` (fallback). It health-checks `/health` and **reuses an already-running server** instead of erroring, so a manually started `python -m api.server` will be adopted by the GUI.
- **`api/server.py`** is the FastAPI app. Its `lifespan` eagerly compiles the LangGraph pipeline, bootstraps FFmpeg, and binds the progress broadcaster to the event loop. Routers live in `api/routes/` (`pipeline`, `config`, `history`, `upload`, `workshop`, `system`, `docs`, `misc`).
- **`src/`** is the Vite/React UI. Tabs in `src/tabs/` (`DocumentaryTab`, `EditorTab`, `HistoryTab`, `DirectUploadTab`, `SettingsTab`) map to the GUI sections.

### Progress: WebSocket with HTTP-polling fallback (important)
On Windows, `localhost` may resolve to IPv6 `::1` while uvicorn binds IPv4 `127.0.0.1`, breaking WebSockets. The system is dual-channel: `api/services/progress_broadcaster.py` keeps a monotonic sequence-numbered (`_seq`) sliding history of events; the React hook `src/hooks/usePipelineWebSocket.ts` uses the WS feed but **falls back to polling `/api/pipeline/progress?after=N`** if the socket drops. Sequence numbers dedupe across both channels. When adding pipeline events, emit through the broadcaster — don't bypass it.

## The LangGraph pipeline (`graph/`)

`graph/pipeline.py` wires all nodes and compiles the graph with a `SqliteSaver` checkpointer (`get_pipeline()` is a process-wide singleton). Nodes live in `graph/nodes/`; the shared state is a `TypedDict` in `graph/state.py`.

Flow: `research → script → script_critic → human_review → (parallel: image_worker + voiceover_worker) → seo → editor_prep → assemble → upload`.

Things to know before editing the graph:
- **Error routing keys on `last_failed_node`, NOT `errors`.** `errors` and `image_paths` are `Annotated[..., operator.add]` accumulators — they grow across checkpoint resumes of the same `thread_id`, so they can't signal "did this node just fail." Each node sets `last_failed_node` to the node name on failure and `""` on success; the `check_*_error` routers and `error_recovery_node` read that field.
- `error_recovery_node` picks an action (`retry` / `fallback` / `skip` / `abort`) and `route_error_recovery` maps it back into the graph. Retrying a parallel worker routes through the `parallel_spawn` dummy fan-out node, not the worker directly.
- The graph compiles with `interrupt_before=["human_review"]`. Human review and the Ghost Editor are **interrupts** — `graph.invoke()` returns early and the run stays paused in SQLite until resumed. `_run_graph_in_background` in `api/routes/pipeline.py` distinguishes "paused at interrupt" (`graph_state.next` is non-empty) from "done" and only emits a completion event in the latter case.
- Runs are keyed by `thread_id = f"run_{run_id}"`. Resuming, retrying, and edits all mutate the same checkpoint thread.

`PipelineRunner` (`core/pipeline_runner.py`) is the thin bridge the GUI and CLI both use: it builds the initial state, creates a per-run output subfolder (`<safe_title>_<timestamp>`), registers a `queue.Queue` with the broadcaster, and kicks off `_run_graph_in_background`.

## Configuration (`config.py` + `core/config_manager.py`)

There are two config layers — don't confuse them:
- **`config.py`** — module-level constants and path resolution (`APP_VERSION`, video dims, FFmpeg discovery, logger factory). `APP_VERSION` must stay in sync with `package.json`, `installer_v4.iss`, and README branding.
- **`core/config_manager.py`** — the runtime settings store (`from core.config_manager import config`). Singleton over `config.json` with **dot-notation** access: `config.get("tts.backend")`, `config.set("pipeline.upload_enabled", True)`. `DEFAULT_CONFIG` in this file is the source of truth for every setting; new defaults are auto-merged into a user's existing `config.json` on load. A human-editable `.env.local` mirror is generated/synced via `ENV_LOCAL_MAP`, and `_validate_v3_fields()` clamps/migrates legacy values.

### Frozen vs. dev paths (critical)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HunterisLive-1/ghost-creator](https://github.com/HunterisLive-1/ghost-creator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
