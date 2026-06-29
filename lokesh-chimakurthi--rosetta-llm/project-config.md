---
trigger: always_on
description: Multi-format bidirectional LLM proxy. Translates between OpenAI Chat Completions, OpenAI Responses, and Anthropic Messages — letting any client SDK talk to any provider regardless of the provider's native API format.
---

# rosetta-llm

Multi-format bidirectional LLM proxy. Translates between OpenAI Chat Completions, OpenAI Responses, and Anthropic Messages — letting any client SDK talk to any provider regardless of the provider's native API format.

## Quick commands

```bash
uv sync                        # install deps + dev tools
uv run pytest tests/ -q        # run tests (18)
uv run ruff check src/ tests/  # lint
uv run ruff format src/ tests/ # format
uv run mypy src/rosetta        # type-check (strict mode)
uv run python -m rosetta       # start the proxy (reads ~/.rosetta-llm/config.json)
```

## Running

```bash
# uvx (no install)
uvx rosetta-llm

# uv tool (persistent)
uv tool install rosetta-llm
rosetta-llm --config /path/to/config.json --port 7860

# Docker
docker run -p 7860:7860 -v ~/.rosetta-llm/config.json:/app/config.json \
  ghcr.io/lokesh-chimakurthi/rosetta-llm:latest
```

## Architecture

**Passthrough fast path**: when `inbound_format == provider.format`, forward verbatim — only the `model` field is rewritten (strip prefix / apply `upstream_name`). No IR allocation.

**Translation slow path**: `inbound bytes → IR → provider format → upstream → provider bytes → IR → inbound format`. Streaming uses per-format state machines that emit/consume `CanonicalStreamEvent`.

**Canonical IR** (`ir/`): every block carries a `_raw` dict sidecar (PrivateAttr) so format-specific fields survive lossy rendering and round-trip correctly. This is critical for reasoning fidelity and tool-call identity preservation.

## Key design decisions

- **uv project** — always use `uv run <cmd>`, never `.venv/bin/python` directly.
- **No pydantic-ai** — pure httpx async. pydantic-ai's agent abstraction obscures wire-level fidelity.
- **No BaseHTTPMiddleware** — blocks SSE streaming. Auth and request-id are pure ASGI middleware.
- **Dockerfile** — multi-stage build using uv. Project is installed into the venv; CMD uses the `rosetta-llm` CLI. Config is mounted at `/app/config.json` via the `ROSETTA_CONFIG` env var.
- **Dockerfile.hf** — Hugging Face Spaces variant. Pulls the GHCR image directly (`FROM ghcr.io/...`) — no clone, no build. Just COPY your config.json in.

## Claude Code gateway

When `ANTHROPIC_BASE_URL` points at Rosetta, Claude Code discovers models via `GET /v1/models` at startup. Rosetta detects Claude Code by the `X-Claude-Code-Session-Id` header and returns a model list tailored for the `/model` picker:

- Models named `claude-*` or `anthropic/*` pass through unchanged.
- All other models get a `claude-code/` prefix — this bypasses Claude Code's built-in filter (which only shows models starting with `claude` or `anthropic`).

On inference, the `claude-code/` prefix is stripped and the model resolves normally. Session headers (`anthropic-beta`, `anthropic-version`, `x-claude-code-session-id`) are forwarded to every upstream call.

## Critical invariants

### Anthropic codec
- **Tool-result ordering**: within a user message, all `tool_result` blocks MUST precede any `text` block. `_enforce_tool_result_ordering()` auto-repairs on render.
- **Tool-use input**: Anthropic's `tool_use.input` is a JSON object. Serialize to `arguments_json_text` (string) in IR via orjson; deserialize back on render.
- **Reasoning signature encoding**: `f"{encrypted_content}@{reasoning_id}"` — split on the **last** `@` to decode. Compaction items prefix with `cm1#`. This is lossless across Anthropic ↔ Responses round-trips.
- **thinking.type**: parsed as `reasoning.thinking_type` (enabled/adaptive/disabled). Rendered back as `thinking: {type, budget_tokens}`.
- **cache_control**: preserved in `_raw` on every content block. Rendered back when present.
- **Tool extras**: `defer_loading`, `type`, `cache_control` from tool `_raw` are merged into rendered tool definitions.

### OpenAI Chat codec
- **System/developer roles**: extracted from message list, concatenated with `\n\n`, stored as `system` on IR.
- **`tool`/`function` role**: each tool message emits one `role=tool` message in the rendered Chat body. Tool results are split from user messages.
- **`file` content part**: degraded to `"[File attached: <name>]"` text placeholder.
- **`input_audio`/`input_video`**: degraded to text placeholder.
- **Stop**: `stop` field accepts string or array. Arrays map to `stop_sequences`; single strings also map.

### OpenAI Responses codec
- **Input items**: message, function_call, function_call_output, reasoning, compaction, item_reference.
- **Output items**: message, function_call, reasoning, compaction.
- **Reasoning**: `encrypted_content` + `id` round-trip via Anthropic signature. `summary` accepts concise/detailed/auto.
- **Phase markers** (commentary/final_answer): preserved in `_raw`, emitted only when the source item had them.

### Pipeline
- **Header forwarding**: `anthropic-beta`, `anthropic-version`, `x-claude-code-session-id` extracted from inbound request and forwarded to every upstream call.
- **Model resolution**: `<provider_key>/<model_name>` split on first `/`. Optional `upstream_name` per model remaps what we forward. `claude-code/` prefix stripped and re-resolved.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lokesh-Chimakurthi/rosetta-llm](https://github.com/Lokesh-Chimakurthi/rosetta-llm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-28 -->
