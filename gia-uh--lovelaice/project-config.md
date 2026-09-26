---
trigger: always_on
description: A sovereign, local-first terminal **coding agent**. The engine is a **native
---

# AGENTS.md — lovelaice

A sovereign, local-first terminal **coding agent**. The engine is a **native
tool-calling ReAct loop** (`Message.tool_calls` on the wire) — one assistant
message → N tool calls → tool results → loop until the model gives a final
answer. Read this before touching the repo.

## Architecture (the native engine)

Everything runs through the `agent/` package. The old Lingo-based agent
(`core.Lovelaice`, `commands/react.py`'s structured-output decide/equip/invoke,
`config.Config`, the Textual `tui/`, `oneshot.py`, `template.py`, `thinking.py`)
was **retired 2026-07-03** — do not look for it.

- `src/lovelaice/cli.py` — typer entrypoint. `lovelaice "prompt"` runs one prompt
  through the engine via an **in-process ACP client** (`acp.client` →
  `acp.server` → coding host). Piped stdin works. Model via `--model` /
  `LOVELAICE_MODEL`. No-arg (tty) prints usage.
- `src/lovelaice/agent/` — the engine.
  - `agent.py` — `Agent` + `AgentConfig` (the user-facing API: `agent.prompt(text)
    -> StopReason`). `AgentConfig` carries model/api_key/base_url/max_tokens and
    the opt-in tool-arg repair knobs.
  - `loops/react_native.py` — `ReActNative`, the canonical loop.
  - `harness.py` — LLM-call seam (`llm_call` → `lingo.LLM.chat(tools=…)`), tool
    dispatch (`execute_tools_batch`), and the `AgentEvent` bus.
  - `tools.py` — `AgentTool` (wraps a `lingo.Tool` + kind/sequential/title) +
    `ToolRegistry` + arg validation.
  - `session.py` / `conversation.py` — persistence; `events.py` — typed events
    (`TurnStart`, `AssistantMessageFinalized`, `ToolExecutionStart/End`, …);
    `hooks.py` — `before_llm_call` / `tool_call` reducer chain; `prompt.py` —
    system-prompt assembly.
- `src/lovelaice/acp/` — ACP surface. **`v1/`** is the current server
  (`v1/server.py` `AcpServerV1` + `v1/__main__.py`), built on the official
  `agent-client-protocol` SDK (`acp.Agent` + `acp.run_agent`); the `lovelaice-acp`
  console script runs it. It speaks **official ACP v1** and supports per-session
  MCP attach, streaming (`AssistantMessageDelta` → `AgentMessageChunk`), token
  `usage` in `PromptResponse`, and `load_session` resume (deterministic
  `~/.lovelaice/acp-sessions/<sid>.jsonl`, `LOVELAICE_SESSIONS_DIR`-overridable).
  The legacy `server.py` (`AcpServer`) / `client.py` (`InProcessAcpClient`) /
  `protocol.py` implement an older hand-rolled "0.1" flat dialect and are
  **frozen** — warden spawns the `AcpServer` class directly and pins
  `lovelaice>=2.6,<3`. See `know-how/acp-v1-server.md`.

  **What "frozen" means, precisely** (qualified 2026-08-01): the freeze exists
  to protect warden, its only consumer. So it forbids **changing or removing
  anything already on the wire** — an existing notification's shape, a method's
  semantics, a field's meaning. It does **not** forbid a purely **additive**
  notification that warden itself needs, since warden cannot be broken by a
  message it did not previously receive.

  Adding one is still a deliberate act, not a loophole. Before you do: confirm
  no existing notification changes, run the whole ACP suite, and check whether
  **v1 already provides it** — v1 is the current server and usually does. If v1
  has it, the honest fix may be to migrate the consumer rather than to extend
  the legacy dialect; weigh that first and say why you chose otherwise.

  Precedent: 2.12.0 added `sessionUpdate: "usage"` here (13 lines, no existing
  branch touched) because warden needed per-turn token cost for the AInBox
  audit log and speaks only this dialect. v1 had already shipped the same data
  in `PromptResponse.usage`, so the real resolution is
  **[warden → ACP v1](https://github.com/syalia-srl/ainbox) migration**, tracked
  in ainbox's `tasks.md`; this was the additive stopgap, chosen deliberately.

## Releasing — `make publish` is a trap, do not use it

Releases go through **CI, not your laptop**. `.github/workflows/release.yaml`
fires on `release: created`, runs the suite, then publishes with `uv publish`
under `permissions: id-token: write` — that is **PyPI Trusted Publishing (OIDC)**,
so there is no token anywhere and none is needed.

The `publish` target in the `makefile` is stale and predates that. It runs
`uv publish --token $(dotenv -f .env get PYPI_TOKEN)`, and no `.env` in this
project has ever carried a `PYPI_TOKEN` — it dies with
`error: a value is required for '--token <TOKEN>'`. Discovered 2026-08-01 while
releasing 2.12.0.

The actual procedure:

```bash
# 1. bump BOTH version strings — they have drifted before (see below)
#    pyproject.toml `version` and src/lovelaice/__init__.py `__version__`
# 2. CHANGELOG entry
# 3. commit named paths, tag, push both
git commit -m "chore: release X.Y.Z" -- pyproject.toml src/lovelaice/__init__.py CHANGELOG.md
git tag vX.Y.Z && git push origin main && git push origin vX.Y.Z
# 4. the release is what publishes — the tag alone does nothing
gh release create vX.Y.Z -R gia-uh/lovelaice --title "vX.Y.Z" --notes-file <notes>
```

**Then verify against PyPI, not against the tag.** A pushed tag and a GitHub
release both exist happily while nothing was published — that is exactly the
state 2.12.0 sat in for a while:

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gia-uh/lovelaice](https://github.com/gia-uh/lovelaice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
