---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development commands

All commands use `uv` (Astral's Python package manager). Install dev dependencies once: `uv sync --extra dev`.

| Task | Command |
|------|---------|
| Launch TUI | `uv run riftor` |
| Lint | `uv run ruff check riftor dev tests` |
| Type check | `uv run pyright riftor` |
| Unit tests | `uv run pytest` |
| Single test | `uv run pytest tests/test_tools.py::test_function_name` |
| Smoke test | `uv run python dev/smoke.py` |
| All CI gates | `make check` (runs lint → typecheck → test → smoke) |
| Build | `uv build` |
| Pre-commit hooks | `make install-hooks` or `uv run pre-commit install` |

CI runs lint, type check, unit tests, and smoke on Python 3.11 + 3.12. All tests run **offline** — no API key or model needed.

- **Offline by design.** The provider checks `RIFTOR_DEMO_RESPONSE` first and, if set, yields that canned text instead of calling litellm (`agent/provider.py`) — this is how the suite and smoke test avoid the network. `tests/conftest.py` supplies the shared fixtures `tmp_workdir`, `engagement`, and `toolctx`. `pytest` runs in `asyncio_mode = "auto"` (no `@pytest.mark.asyncio` needed).
- `dev/smoke.py` drives the *real* Textual app headlessly end-to-end and cancels any live stream, so it exercises the UI without a model. Extend it when adding TUI behavior.

## Architecture

riftor is a Python 3.11+ TUI pentest assistant: a Textual full-screen app backed by litellm, organized around the **RIFT** methodology (Recon → Intrusion → Foothold → Takeover).

### Entry and dispatch (`riftor/__main__.py`)

CLI arg parsing → loads config → dispatches:
- `--config`: prints config path and exits
- `--doctor`: checks installed recon tools on `PATH` and exits
- `--prompt`/`-p` (a.k.a. `--headless`): runs `headless.py` (one-shot, non-interactive; also reads stdin)
- default: launches `tui/app.py` (the full Textual app)

Other flags apply to either path: `--version`, `--model`, `--api-key`, `--workdir`, `--scope-file`, and `--i-know-what-i-am-doing-give-me-full-access` (stored as `yolo`; see below).

### Agent loop (shared by TUI and headless)

The core loop lives in both `tui/app.py` and `headless.py`. The skeleton is the same; the **gating differs by mode** (see "TUI vs headless" below). Each iteration:

1. User input → added to `Context` (conversation history)
2. `Context.repair()` runs first — it ensures every assistant `tool_call` has a following tool result, inserting a synthetic `[interrupted: …]` result for any orphaned id (this is what keeps a cancelled/crashed turn replayable)
3. `Provider.stream_turn(messages, tools)` streams `("text", delta)` chunks as the model talks, then yields a final `("done", Turn)`. **Tool calls are buffered during streaming and only appear on the final `Turn`** — they are reassembled by `index` from the chunk fragments, not streamed live.
4. For each tool call: scope-sensitive tools are checked against the engagement scope; dangerous tools (`requires_permission`) go through the `Permissions` engine
5. Tool results → context; loop continues up to `max_steps` (default **16**, `config.py`) or until a turn has no tool calls

### Sub-packages

- **`riftor/agent/`** — LLM abstraction. `provider.py` wraps litellm (streaming, tool calling, retry/backoff with deterministic jitter, `classify_error()` mapping exceptions to a typed `ProviderError`). **litellm is lazily imported** on first model call (~2.4s — an estimate in a comment, not a measured constant) to keep startup fast. `context.py` builds the system prompt from `agent/prompts/system.md` (loaded via `importlib.resources`) and appends `LORE_PREAMBLE` when `lore=True`; it also handles `compact()` (clip old tool results to ~400 chars, keep the last ~8 messages) and the `repair()` above. `session.py` persists sessions as JSON via tmp-file + `Path.replace()`; the `complete` flag marks mid-turn checkpoints so a crashed run can be detected and resumed.
- **`riftor/providers.py`** — provider registry feeding config and the model picker. Holds the `PROVIDERS` table and `PROVIDER_DEFAULTS` (curated model lists), resolves a provider from a model id (`provider_key_for_model`), and `fetch_models()` queries the live model list but **degrades to the curated defaults on any network/auth failure**. Edit this when touching model selection or credential resolution.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Estudely/riftor](https://github.com/Estudely/riftor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-13 -->
