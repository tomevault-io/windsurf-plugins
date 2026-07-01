---
trigger: always_on
description: **decode** is a terminal **coding agent** ("agentic harness") built from scratch, step by step, as an educational open-source course. It is a single Python package, `decode`, exposing a TUI you run in your terminal: a Pydantic-AI ReAct loop driving file/bash/web/MCP tools, with pluggable inference (Gemini / OpenRouter / Modal), local + remote sandboxing, Opik observability, and a Kitaru durability runtime. Standalone single-package Python (`cli-tool-python` shape); the TUI is a module *inside* t
---

# decode

**decode** is a terminal **coding agent** ("agentic harness") built from scratch, step by step, as an educational open-source course. It is a single Python package, `decode`, exposing a TUI you run in your terminal: a Pydantic-AI ReAct loop driving file/bash/web/MCP tools, with pluggable inference (Gemini / OpenRouter / Modal), local + remote sandboxing, Opik observability, and a Kitaru durability runtime. Standalone single-package Python (`cli-tool-python` shape); the TUI is a module *inside* the package (`prompt_toolkit` input + `Rich` output), not a separate service.

License: **Apache-2.0**. Depth references below name **squid scaffold specs** (shipped in the `iusztinpaul/squid` plugin, not this repo) — read them via the plugin cache.

# Key Components

Single package — one bullet for the package, then the internal module map under [Project Structure](#project-structure). Modules are built incrementally as the course progresses; only `config/`, `entities/`, and `logging.py` are foundational from day one.

- **`decode`** — [`src/decode/`](src/decode/): the whole coding agent. Python 3.12+, `cli-tool-python` shape (Click entrypoint launching the TUI). Conventions: async I/O for network/DB, sync for CPU; infrastructure imported directly (no premature interfaces); shared models in `entities/`, narrow types in `<module>/types.py`; every entrypoint calls `init_logger()` at module level before any project import. Depth: squid spec `python-backend` + `cli-tool-python`.

# Project Structure

The intended target tree. Most `src/` subpackages are created **when you reach their step** — do not pre-create empty packages. `tests/` mirrors `src/` 1:1.

```
.
├── AGENTS.md / CLAUDE.md          # this memory file (+ Claude Code import)
├── pyproject.toml                 # uv + hatchling; deps grow per step
├── Makefile                       # install / test / lint / format / pre-commit / build / ci
├── .pre-commit-config.yaml        # format + lint (commit) · unit tests (push)
├── .env.example                   # config & secrets surface
├── docs/
│   ├── adr/                       # Architecture Decision Records (Nygard)
│   └── glossary.md                # ubiquitous language
├── tasks/                         # file-based tracker — one md per task
├── tests/{unit,integration}/      # unit mirrors src/ 1:1; integration touches real infra
└── src/decode/
    ├── __init__.py
    ├── logging.py                 # init_logger() — module-level in every entrypoint
    ├── cli.py                     # Click entrypoint → launches the TUI        [bootstrap]
    ├── config/settings.py         # pydantic-settings; module-level `settings` singleton
    ├── entities/                  # shared models: Message, Conversation, ToolCall, Task…
    ├── tui/                       # input: prompt_toolkit · output: Rich (answers via SSE)
    ├── harness/                   # message Queue + Priority Gate around the loop
    ├── agent/                     # Pydantic-AI ReAct loop (LLM ⇄ Tools)
    ├── agents/                    # agents catalog (Build/Plan/Explore/Code-Reviewer) + subagents
    ├── tools/                     # file I/O, Bash, web, tasks, MCP factory, skill dispatcher, LSP, AskUser
    ├── permissions/               # allow/ask/deny · modes (default/plan/edit/bypass) · settings.json
    ├── sandbox/                   # Bash execution — local (Docker/Firecracker) + remote (Modal)
    ├── services/lsp/              # LSP Service — hand-rolled stdio client; FIRST concrete services/ entry (ADR-0007)
    ├── services/                  # services interface: LLM gateway, memory, MCP servers land here later
    ├── runtime/                   # Kitaru: credentials proxy, durability, scheduling, HITL
    ├── context/                   # context engineering: compaction + conversation log (JSONL)
    ├── memory/                    # AGENTS.md / MEMORY.md loading
    └── observability/             # Opik tracing
```

**Scripts & entrypoints.** Operator scripts in `scripts/`; CLI entrypoint declared in `pyproject.toml` `[project.scripts]` (`decode = "decode.cli:cli"`). **Every entrypoint module calls `init_logger()` at module level before any project import.**

# Tech Stack

Single Python toolchain — `uv`, `ruff`, `pytest`. **Python 3.12+.**

| Layer | Choice | Notes |
|---|---|---|
| Package/deps | `uv` (+ `hatchling` build) | `uv.lock` committed; `uv sync` is the installer. |
| Lint/format | `ruff` | One config block in `pyproject.toml`; format + check are separate passes. |
| Test | `pytest` (+ `asyncio`, `mock`) | `tests/` mirrors `src/`; `filterwarnings=["error"]`. |
| CLI / TUI | `click` · `prompt_toolkit` · `rich` | Click wrapper is thin; logic in pure functions. |
| Agent loop | `pydantic-ai` | ReAct loop (LLM ⇄ tools). *added at its step* |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [decodingai-magazine/building-a-coding-agent-from-scratch-course](https://github.com/decodingai-magazine/building-a-coding-agent-from-scratch-course) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
