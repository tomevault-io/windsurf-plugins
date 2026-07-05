---
trigger: always_on
description: Project-level guidance for Claude Code agents working on this repository. The user's global `~/.claude/CLAUDE.md` still applies; this file is additive and takes precedence on conflicts.
---

# CLAUDE.md — Animus

Project-level guidance for Claude Code agents working on this repository. The user's global `~/.claude/CLAUDE.md` still applies; this file is additive and takes precedence on conflicts.

## North Star

Animus is a **local-first agentic coding tool** powered by small local LLMs via llama-cpp-python. The capability ladder we're building toward:

1. **Simple ops, 100% accurate.** Move a file, rename a folder, mkdir, copy, delete — these must never be wrong. Small models miscall `mv`/`rm` when routed through `bash`, so each of these belongs as a dedicated tool with structured args and workspace boundary checks.
2. **Multi-step construction.** Given a prompt like "build me a CLI that does X," Animus should: deconstruct the task into elementary components → work each sequentially with appropriate tools → reconstruct the whole → devise and run tests. The existing `src/core/planner.py` is the seed but is **not yet wired into the runtime** — that's queued work.

Every new feature should advance one of those two rungs. If a change doesn't help simple-op reliability or multi-step construction, it's probably not for Animus.

## Critical principle: validate against a real GGUF

The unit tests mock the provider. They missed three real bugs in the v2-polish PR:

1. Installed CLI entry point bypassed Typer.
2. NANO `grammar_mode="full"` wedged the ReAct loop — model could never produce a final text answer.
3. Most GGUFs lack `general.parameter_count` metadata, collapsing tier detection to NANO.

**Never merge a change that touches the runtime, providers, grammar, tier behavior, or tool dispatch without a real-GGUF run.** Use `animus --debug` to capture a full trace. Inspect `.animus/trace/<session-id>.jsonl`. The user keeps GGUFs at `C:\Users\charl\.animus\models\`.

## Architectural conventions

- **Provider protocol** (`src/providers/base.py`) abstracts LLM backends. Tool-call recovery from raw model output lives in `src/providers/parsing.py` and is provider-agnostic — small models emit JSON in fenced blocks, `<tool_call>` tags, embedded prose, etc.; the shared parser handles all of those.
- **Tool registration** is declarative. Add tools by appending to `src/tools/defaults.py:register_default_tools`. Each tool is `(args: dict, workspace: Workspace) -> ToolResult` with a JSON schema, permission level, and min tier.
- **Tier system** (`src/core/tiers.py`) scales grammar enforcement, planner usage, max turns, and tool count to model size. NANO and SMALL use `grammar_mode="first_turn"` — never use `"full"` again; it makes loop termination impossible.
- **Workspace boundary** is enforced at every tool entry point via `workspace.resolve(path)`. Never bypass.
- **Observability**: the ReAct loop emits structured events (turn_start, iteration_start, provider_response, tool_call, tool_result, etc.) via `src/observability/tracer.py`. `animus --debug` writes JSONL + Rich live view; the JSONL is the source of truth and the pretty view is derived. **When adding runtime features, emit events for them** — it's how we'll catch the next real-model loop pathology.

## Conventions for new tools

When you add a new tool (e.g. `move_file`, `mkdir`):

1. Handler in `src/tools/<area>.py`: signature `def handle_X(args, workspace) -> ToolResult`. Resolve every path through `workspace.resolve()`. Catch `OSError` and return `ToolResult(output=..., is_error=True)`.
2. Register in `src/tools/defaults.py` with a JSON schema, `PermissionLevel`, and `min_tier`. Simple ops (move, rename, mkdir) should be `NANO`-tier minimum so all model sizes can use them.
3. Add a test file `tests/test_tools_<area>.py` exercising: happy path, workspace-escape path, missing-source path, OSError path.
4. Run the suite (`pytest`) **and** a real-GGUF run with `--debug` against a small workspace that uses the new tool.

## Conventions for runtime/provider changes

- Keep the runtime's per-iteration cost low. Small models on CPU regenerate the whole prompt every turn; anything you do per-iteration multiplies. Prefer cache-once-per-turn (see `cached_grammar` in `runtime.py`).
- Tool outputs are truncated to `config.session.max_tool_output_chars` before they re-enter the prompt. The **debug trace sees the full, untruncated output** — emit the tracer event before the truncation, not after.
- The `Provider.estimate_tokens` protocol method should use the real tokenizer when available; only fall back to chars/4 if it raises. Inaccurate counts over-fill small context windows.

## Out of scope

- Audio / TTS / voice. The v2 rewrite already removed these; do not re-add.
- Cloud/HTTP providers. Local-first is the point.
- Anything that depends on internet at runtime (huggingface-cli installs are fine ahead of time; runtime model loading is purely local).

## Open follow-on work (ranked)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crussella0129/Animus_1.0](https://github.com/crussella0129/Animus_1.0) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-05 -->
