---
trigger: always_on
description: This guide is for AI coding agents working on the OmicsClaw codebase.
---

# AGENTS.md — OmicsClaw Guide for AI Coding Agents

This guide is for AI coding agents working on the OmicsClaw codebase.

## Repository Working Contract

Before any complex repository maintenance, feature, or refactor task, read
`README.md` first for project context and prior decisions. Then read this
`AGENTS.md`, root `SPEC.md`, and the directly relevant code/docs.

Core rules:

- Reply in the user's language, usually Chinese or English.
- Stay concise, practical, and execution-focused.
- Use `docs/superpowers/playbooks/` as the repository's on-demand workflow
  guidance for debugging, TDD, planning, parallelization, verification,
  code review, and branch completion.
- Treat those playbooks as binding workflow guardrails with iron laws, red
  flags, and required evidence, not as optional summaries.
- When you make an important decision or complete a meaningful milestone,
  update `README.md` while preserving its existing structure.

## Project Overview

OmicsClaw is a multi-omics analysis platform supporting 5 domains: spatial transcriptomics, single-cell omics, genomics, proteomics, and metabolomics. Each skill is a self-contained module that performs a specific analysis task via CLI or Python API. All processing is local-first. Design is inspired by [ClawBio](https://github.com/ClawBio/ClawBio).

**Note**: OmicsClaw evolved from SpatialClaw and now uses a unified `omicsclaw.py` entrypoint.

## Setup

```bash
cd /data1/TianLab/zhouwg/project/OmicsClaw
pip install -e .

# Add optional extras only when needed
# pip install -e ".[interactive]"
# pip install -e ".[tui]"
# pip install -e ".[memory]"
# pip install -e ".[full]"

python omicsclaw.py list   # or: oc list
python omicsclaw.py run spatial-preprocess --demo
```

> **`oc` short alias**: After `pip install -e .`, both `omicsclaw` and `oc` commands
> are available system-wide. `oc` is registered via `[project.scripts]` in
> `pyproject.toml` and points to `omicsclaw.cli:main` — the same entry point as
> `omicsclaw`. No PATH tricks needed.
>
> **Dependency source of truth**: Root dependency management lives in
> `pyproject.toml`. The repository does not use a root `requirements.txt` as a
> primary install entrypoint.

## Commands

> Both `python omicsclaw.py <cmd>` and the short alias `oc <cmd>` work identically
> after `pip install -e .` (or `make install-oc`).

| Command | Purpose |
|---------|---------|
| `oc list` | List all 50+ skills across 5 domains |
| `oc run <skill> --demo` | Run a skill with demo data |
| `oc run <skill> --input <file> --output <dir>` | Run with user data |
| `oc interactive` | **Start interactive terminal chat (CLI mode)** |
| `oc interactive --ui tui` | **Start full-screen Textual TUI** |
| `oc interactive -p "<prompt>"` | **Single-shot mode (non-interactive)** |
| `oc interactive --session <id>` | **Resume a previous session** |
| `oc tui` | Alias for `interactive --ui tui` |
| `oc app-server` | Start the FastAPI backend used by OmicsClaw-App / web frontends |
| `oc mcp list` | List configured MCP servers |
| `oc mcp add <name> <cmd> [args]` | Add an MCP server |
| `oc mcp remove <name>` | Remove an MCP server |
| `oc mcp config` | Show MCP config file path |
| `oc onboard` | Run interactive setup wizard for LLM, runtime, memory, and channels |
| `python -m pytest -v` | Run all tests |
| `make test` | Alias for pytest |
| `make demo` | Run preprocess demo |
| `make install-oc` | (Re)install package + activate `oc` alias |
| `make oc-link` | Quick wrapper script in `~/.local/bin/oc` (no pip) |
| `make bot-telegram` | Start Telegram bot |
| `make bot-feishu` | Start Feishu bot |

## Project Structure

```
OmicsClaw/
├── omicsclaw.py                # Main CLI runner (SKILLS dict, DOMAINS registry)
├── omicsclaw/                  # Core framework (domain-agnostic)
│   ├── common/                 # report.py, session.py, checksums.py
│   ├── core/                   # registry.py, dependency_manager.py
│   ├── loaders/                # File-extension → domain detection helpers
│   ├── memory/                 # Graph memory system
│   ├── routing/                # Multi-agent routing
│   ├── agents/                 # Agent definitions
│   └── interactive/            # Interactive CLI/TUI package
│       ├── __init__.py         # Package entry: run_interactive(), main()
│       ├── _constants.py       # Banner, LOGO, slash commands, slogans
│       ├── _session.py         # SQLite session persistence (aiosqlite)
│       ├── _mcp.py             # MCP server config / YAML management
│       ├── interactive.py      # prompt_toolkit REPL loop (CLI mode)
│       └── tui.py              # Textual full-screen TUI (TUI mode)
├── skills/                     # Domain-organized skills + shared utilities
│   ├── spatial/                # 15 spatial transcriptomics skills
│   │   ├── _lib/               # ★ Shared spatial utilities (adata_utils, viz, loader, etc.)
│   │   │   ├── viz/            # Unified visualization package (13 modules)
│   │   │   ├── adata_utils.py  # AnnData helper functions
│   │   │   ├── loader.py       # Multi-platform data loader
│   │   │   ├── dependency_manager.py  # Lazy import manager
│   │   │   ├── exceptions.py   # Domain-specific exceptions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TianGzlab/OmicsClaw](https://github.com/TianGzlab/OmicsClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
