---
trigger: always_on
description: Offline voice agent framework for robots and desktop apps. Pure-Python package, no cloud dependencies, runs on CPU/CUDA/Metal.
---

# EdgeVox — Claude Code Rules

Offline voice agent framework for robots and desktop apps. Pure-Python package, no cloud dependencies, runs on CPU/CUDA/Metal.

## Project layout

```
edgevox/
├── edgevox/            # Source package
│   ├── audio/          # VAD, mic capture, playback
│   ├── stt/            # STT backends (faster-whisper, sherpa-onnx)
│   ├── llm/            # llama.cpp / Gemma integration
│   ├── tts/            # TTS backends (Kokoro, Piper, Supertonic, PyThaiTTS)
│   ├── core/           # Pipeline orchestration
│   ├── cli/            # CLI entrypoints
│   ├── ui/             # TUI widgets
│   ├── integrations/   # ROS2 bridge, etc.
│   ├── tui.py          # Main TUI app
│   └── setup_models.py # Model downloader
│   ├── server/         # FastAPI web UI + WebSocket server
├── webui/              # React frontend (Vite + Tailwind)
├── scripts/            # Utility scripts (model upload, etc.)
├── voices/             # Voice config files
├── docs/               # Project docs
├── website/            # VitePress site
└── pyproject.toml
```

Entrypoints (see `pyproject.toml`):
- `edgevox` → `edgevox.tui:main` (TUI default, `--web-ui` for web, `--simple-ui` for CLI)
- `edgevox-cli` → `edgevox.cli.main:main`
- `edgevox-setup` → `edgevox.setup_models:main`

## Supported languages & backends

| Language | STT | TTS |
|----------|-----|-----|
| English, French, Spanish, etc. | faster-whisper | Kokoro |
| Vietnamese | sherpa-onnx (zipformer) | Piper |
| German, Russian, Arabic, Indonesian | faster-whisper | Piper |
| Korean | faster-whisper | Supertonic |
| Thai | faster-whisper | PyThaiTTS |

Models are hosted on `nrl-ai/edgevox-models` (HuggingFace) with fallback to upstream repos.

## Architecture principles

- **Plug-and-play, customizable by default.** Every component — STT backend, TTS backend, LLM, VAD, agent loop behavior, pipeline stage, tool, skill, hook — must be swappable without editing core code. Prefer Protocols, registries, and decorators over hard-coded paths. New behavior lands as a new plugin/hook/backend, not as a patch to an existing module. If you find yourself adding a conditional to core for a specific use case, step back and extract it into an injection point instead.

## Agent harness architecture

The agent harness (`edgevox/agents/` + `edgevox/llm/hooks_slm.py` + `edgevox/llm/tool_parsers/`) is fully documented under `docs/documentation/`:

- [`agent-loop.md`](docs/documentation/agent-loop.md) — the six-fire-point loop, parallel dispatch, handoff short-circuit.
- [`hooks.md`](docs/documentation/hooks.md) — hook authoring contract, built-ins, ordering rules.
- [`memory.md`](docs/documentation/memory.md) — `MemoryStore` / `SessionStore` / `NotesFile` / `Compactor`.
- [`interrupt.md`](docs/documentation/interrupt.md) — barge-in signals + cancel-token plumbing.
- [`multiagent.md`](docs/documentation/multiagent.md) — Blackboard, BackgroundAgent, AgentPool.
- [`tool-calling.md`](docs/documentation/tool-calling.md) — parser chain + grammar-constrained decoding roadmap.

### Harness rules

- **Typed `AgentContext` fields** (`ctx.tool_registry`, `ctx.llm`, `ctx.interrupt`, `ctx.memory`, `ctx.artifacts`, `ctx.blackboard`) are the public plumbing surface. `ctx.state` is user-only scratch — framework code must not write magic keys there.
- **Hook-owned state** lives under `ctx.hook_state[id(self)]`. Keying by `id(self)` is what guarantees two instances of the same hook class don't share state.
- **Barge-in is enforceable, not advisory.** Every `LLM.complete` call threads `ctx.interrupt.cancel_token` via `stop_event=…` so llama-cpp's `stopping_criteria` actually halts generation within one decode step.
- **Tokenizer-exact token counts.** `estimate_tokens(messages, llm)` and `LLM.count_tokens` replace the `chars // 4` heuristic when an LLM is available. Required for correct context-window decisions on CJK / Vietnamese / Thai.
- **Tool-call parsing runs raw-first.** `parse_tool_calls_from_content` tries detectors against the raw content before stripping `<think>` blocks — Qwen3 emits tool calls inside reasoning blocks (see [llama.cpp#20837](https://github.com/ggml-org/llama.cpp/issues/20837)).
- **Preset parsers are validated at load.** `resolve_preset(slug)` asserts every name in `tool_call_parsers=(...)` is a registered detector; a typo fails loudly rather than silently disabling detection.
- **Model-emitted tool-call ids round-trip.** Mistral's `[TOOL_CALLS]` format carries a 9-char id that the follow-up `role="tool"` message must reuse. `ToolCallItem.id` plumbs this through the parser chain and the agent loop.

### Preferred import surfaces

- Agent framework: `from edgevox.agents import LLMAgent, AgentContext, Session, Handoff, ...`
- Built-in hooks: `from edgevox.agents.hooks_builtin import MemoryInjectionHook, TokenBudgetHook, ...`
- SLM hardening: `from edgevox.llm.hooks_slm import default_slm_hooks`
- Memory: `from edgevox.agents.memory import JSONMemoryStore, NotesFile, Compactor, estimate_tokens`
- Multi-agent: `from edgevox.agents.multiagent import Blackboard, BackgroundAgent, AgentPool`
- Interrupt: `from edgevox.agents.interrupt import InterruptController, InterruptPolicy, EnergyBargeInWatcher`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nrl-ai/edgevox](https://github.com/nrl-ai/edgevox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
