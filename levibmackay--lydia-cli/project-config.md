---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Lydia is a local AI coding agent CLI — a personal, API-key-free alternative to
Claude Code / Cursor, built on top of a local [Ollama](https://ollama.com)
daemon, with an optional FastAPI server (`server/`) so Ollama can run on a
more powerful remote machine instead. It is a portfolio project for Levi
(CS student), so code quality, tests, and a clean architecture matter more
than shipping speed.

This is a **two-package monorepo**: `src/lydia` (the CLI, always needed)
and `server/lydia_server` (optional, a separate installable package that
depends on `lydia` as a library). Both are typically installed into one
shared venv for local dev.

## Commands

```bash
# Install both packages (editable) into one shared venv
.venv/bin/pip install -e ".[dev]"
.venv/bin/pip install -e "server/[dev]"

# Run the CLI package's test suite (140 tests)
.venv/bin/pytest
.venv/bin/pytest tests/test_agent_loop.py                                   # one file
.venv/bin/pytest tests/test_agent_loop.py::test_tool_call_then_final_answer # one test

# Run the server package's test suite (14 tests) — has its own pyproject.toml,
# so run it from server/, not the repo root
cd server && ../.venv/bin/pytest

# Run the CLI itself (after install -e, `lydia` is also on PATH via a symlink
# into /opt/homebrew/bin)
lydia                      # interactive chat REPL in the current project
lydia ask "question"       # one-shot, no tools, good for smoke-testing the LLM layer
lydia analyze              # project scanner output
lydia config show

# Run the server locally (needs a LYDIA_SERVER_TOKEN or it refuses to start)
LYDIA_SERVER_TOKEN=dev-token .venv/bin/lydia-server
```

There is no separate lint/format command configured yet.

### Testing against the real Ollama daemon

Unit tests never touch the network (`httpx.MockTransport` for the LLM client,
tmp_path repos for git/filesystem tools) — `pytest` should never require
Ollama to be running. To manually exercise the real thing:

```bash
ollama list                 # confirm a model is pulled; qwen3.5 variants are what's tested locally
lydia config set think off  # qwen3 is a thinking model; off = much faster manual testing
```

When testing the agent loop end-to-end (tool calls + confirmation prompts),
piping input via `printf ... | lydia` is unreliable — Rich's `Confirm.ask`
and `prompt_toolkit` fight over a non-tty stdin and the confirm dialog will
spuriously EOFError (it fails *safe*, i.e. auto-declines, so this looks like
a bug but isn't one). If you need to script an end-to-end test of a
confirmation flow, drive it through a real pty (see git history around the
Milestone 3 commit for a working example using Python's `pty` module), not a
plain pipe. `lydia ask "..." --yes` sidesteps this entirely for scripted
end-to-end checks since it never needs a y/n prompt in the first place.

To manually verify the client/server split end-to-end (not just
`server/tests/`'s fake-provider unit tests): run a real server locally
against the real local Ollama, point a `lydia` project config at it, and
confirm both that it works *and* that the server's own log only shows
`/v1/chat`/`/v1/models` traffic — never any file access — which is the
actual proof that tool execution stayed client-side:

```bash
LYDIA_SERVER_TOKEN=dev-token .venv/bin/lydia-server &
lydia config set server_url http://127.0.0.1:8000 --project
lydia config set api_key dev-token --project
lydia ask "read some_file.py and summarize it" --yes
```

## Architecture

Layering, outer to inner — each layer only depends on the ones below it:

```
cli/     Typer commands + Rich rendering + prompt_toolkit REPL   (depends on: agent, llm, config, context)
agent/   orchestration: system prompt, tool registry, the loop   (depends on: llm, tools, config)
tools/   pure functions that touch the filesystem/shell/git      (depends on: nothing else in lydia)
llm/     ModelClient protocol + OllamaClient + RemoteClient       (depends on: nothing else in lydia)
context/ project scanner + semantic index (chunk/embed/search)   (depends on: llm (embeddings), database
database/ SQLite storage for the semantic index                  (depends on: nothing else in lydia)
config/  layered JSON settings                                   (depends on: nothing else in lydia)

server/  (separate package, lydia_server/) — FastAPI inference proxy.
         Depends on lydia as a library (reuses OllamaClient directly as
         its provider). Never touches tools/, agent/, or cli/ — tool
         execution always stays client-side. See server/README.md.
```

`llm/` is two concrete clients behind one structural interface
(`llm/protocol.py::ModelClient`): `OllamaClient` talks to a local Ollama
daemon directly, `RemoteClient` talks to a `server/` instance over HTTPS.
Everything above `llm/` — `agent/loop.py`, `agent/tools.py`,
`context/indexer.py`/`retriever.py` — type-hints against `ModelClient`,
never a concrete class, and is handed whichever one `llm/factory.py::build_client`
constructs based on `config.server_url`. This is *the* seam that makes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [levibmackay/lydia-cli](https://github.com/levibmackay/lydia-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-13 -->
