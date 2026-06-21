---
trigger: always_on
description: Guidance for AI coding agents (Codex, Cursor, Copilot, Claude Code, Gemini CLI, etc.)
---

# AGENTS.md

Guidance for AI coding agents (Codex, Cursor, Copilot, Claude Code, Gemini CLI, etc.)
working **on** the Second Brain codebase. Humans should read [README.md](README.md);
this file is the agent-facing companion. If anything here conflicts with an
explicit user instruction, the user wins.

## ⚡ This is a microkernel (the kernel)

Second Brain is a **microkernel**: a minimal core that boots, runs the
agent turn, persists conversations, and loads plugins. Most product capability
arrives through installed packages. **Read the "THE KERNEL" section at the top
of [CLAUDE.md](CLAUDE.md) before making changes.** Search, scheduling commands
and subagents, integrations, Telegram, file-editing tools, shell tools, memory
tooling, and heavy parsers are package capabilities unless the runtime catalog
proves they are installed.

## What this project is

Second Brain is a **local-first AI kernel**: a
programmable conversation runtime with SQLite persistence, an agent turn loop,
five plugin families, package install/uninstall, and a live plugin watcher.
Full-product capabilities such as indexing/retrieval, durable memory workflows,
web search, scheduling workflows, integrations, and Telegram belong in the
store; the lightweight Timekeeper event clock stays in the kernel. Python +
SQLite. Solo-maintained (Henry). There is **no GUI** — a Flet GUI was removed;
do not reintroduce one.

## Setup commands

```bash
pip install -r requirements.txt   # Python 3.11+ required
python main.py                    # run the app (delegates to main.pyw)
python -m pytest -q --basetemp .pytest_tmp_full   # run the test suite
python -m pytest tests/test_service_llm.py -q --basetemp .pytest_tmp_llm
```

There is no build step and no linter config in the repo. Match the surrounding
style rather than reformatting.

## Environment notes

- **Platform is Windows-first** (developed on Win11). Paths in config are
  Windows-style; the code is cross-platform but test on Windows assumptions.
- The default dev shell is **PowerShell**. In PowerShell use `$null`, `$env:VAR`,
  and backtick line-continuation — not bash syntax.
- The app stores state in a per-OS **DATA_DIR** (see [paths.py](paths.py)):
  `%LOCALAPPDATA%/Second Brain/` on Windows. Config lives in `config.json` /
  `plugin_config.json` there; the SQLite DB, sandbox plugins, installed plugins,
  and optional `memory.md` live there too.
  Don't hardcode DATA_DIR — import from `paths.py`.
- An LLM profile is required for agent features. Tests stub the LLM, so they
  run without API keys.

## Architecture (where things live)

- `state_machine/` — pure conversation primitives: participants, turns, phases,
  actions, forms, approvals, serializable `PhaseFrame`s. Keep this layer pure
  (no transport, no I/O coupling).
- `runtime/` — sessions, persistence, approvals, dispatch, agent-turn driver
  (`conversation_loop.py`), and `context.py` (`SecondBrainContext`, the bag every
  plugin receives). `conversation_runtime.py` is the single dispatcher and is the
  accepted "ugly duckling" — large on purpose.
- `agent/` — `system_prompt.py` (the only place the system prompt is assembled;
  sections are gated by which tools the active scope exposes), `tool_registry.py`.
- `plugins/` — base contracts plus the small kernel plugin set.
- `pipeline/` — file watcher, SQLite task queue, orchestrator DAG; it idles
  until task packages are installed.
- `events/` — pub/sub bus.
- `templates/` — the **source of truth** for how to author each plugin family.

## The plugin system (read this before adding features)

Everything user-extensible is a plugin in one of five families. Each family has
a built-in dir, a sandbox dir in DATA_DIR, and an installed-package dir in
DATA_DIR. Discovery is by file presence and filename prefix.

| Family   | Base class      | Built-in dir         | Installed dir                  | File prefix   |
|----------|-----------------|----------------------|--------------------------------|---------------|
| Tools    | `BaseTool`      | `plugins/tools/`     | `installed_plugins/tools/`     | `tool_*.py`   |
| Tasks    | `BaseTask`      | `plugins/tasks/`     | `installed_plugins/tasks/`     | `task_*.py`   |
| Services | `BaseService`   | `plugins/services/`  | `installed_plugins/services/`  | `service_*.py`|
| Commands | `BaseCommand`   | `plugins/commands/`  | `installed_plugins/commands/`  | `command_*.py`|
| Frontends| `BaseFrontend`  | `plugins/frontends/` | `installed_plugins/frontends/` | `frontend_*.py`|

Rules of thumb:

- **Tool** = an LLM-callable action returning a `ToolResult`. Put the
  model-facing summary in `llm_summary`; put frontend/debug payload in `data`.
  Tools declare `requires_services` and are gated on those services being loaded.
- **Task** = pipeline/event worker; should be idempotent.
- **Service** = a long-lived shared backend with `_load()`/`unload()` lifecycle
  and a top-level `build_services(config) -> dict` factory. Reach peers via
  `self.services`. A service that needs runtime objects implements
  `bind_runtime(*, tool_registry=None, orchestrator=None, command_registry=None,
  frontend_manager=None, runtime=None)` — see `service_plugin_watcher.py`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [henrydaum/second-brain-art](https://github.com/henrydaum/second-brain-art) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-21 -->
