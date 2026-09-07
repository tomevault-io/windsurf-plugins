---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Source of truth

`DESIGN.md` is the single source of truth. Section 0 lists closed decisions — do not reopen them; **section 19 amends some of them** (executor dir without `.git`, Grok pull mode cut) and **table 19.9 is the only authoritative phase plan** (older plan sections are history). If the implementation diverges from the doc, fix the doc in the same commit and bump its revision number. The doc is Polish; user-facing docs and README are English first.

## Repository state

Obsidian Phase B (`vault.py`: context, decisions sync, inbox, Claudian kit), `/council:analyze` (`analyze.py`), delegation policy + session budget (`policy.py`, DESIGN §22), quota/no-response fallback to `cheap` with cooldown (§23) — all done 2026-09-05, tag v1.0.0-rc1. Install/ops layer done (2026-09-05, DESIGN §21): marketplace install, SessionStart hook (`council session-start`), `council setup/doctor` with login checks (`setup.py`), privacy check in CI, Obsidian bridge Phase A (`obsidian.py`, DESIGN §20). Phase 3a done (2026-09-05): user docs in `docs/` (EN), `council report`; first real epic run through the MCP tools on `WORKSPACE/sitek-site` (business-card site: Codex assets, Copilot copy, Antigravity page). Phase 2b done: 17 MCP tools (`council_models/ask/probe/plan/dispatch/status/answer/cancel/review/verdict/merge/handoff/defect/stats/why/compare/playbooks`), `council init/doctor/events` CLI, agents `council-planner/reviewer/integrator`, `UserPromptSubmit` hook, gates, trust per model in `.council/stats.json` (§19.13), `LESSONS.md` injected into TASK.md, playbooks in `playbooks/` (+ user `.council/playbooks/`), `dissent` in reports, routing by task type incl. `assets` (§19.12). Verified live (Phase 1): Codex + local Ollama in parallel → `review`, blocked → answer → resume. Unit-tested only: merge, conflict re-dispatch, reject → attempt+1 → failed at 3. Not yet / deferred past v1.0: repo publication (user decision), `council bench/night`, profiles, `council_recall`, `/council:analyze|spec|architect|docs`, solo-vs-council estimate, remaining playbooks. Environment: `docs/probe-2026-09.md`; lessons: DESIGN §19.11.

## What this is

`super-claude-code` — a Claude Code plugin named `council` plus a Python MCP server `council-mcp` (package `council_mcp`). Claude Code plans, delegates, reviews and merges; other providers' models (Ollama on remote srv-ai, Gemini CLI, Codex CLI, Grok Build CLI, `claude -p` cheap model) execute disjoint tasks in parallel in the same repo, each in its own git worktree + branch `council/<id>`. MIT, public project; company (NUCO) specifics live only in `profiles/nuco.json`, `playbooks/merit-integration.json`, `examples/nuco-wms/` — never in the core.

## Commands

```bash
uv sync                                  # install (dev group included)
uv run pytest -q                         # all tests (respx mocks; nothing hits a live model)
uv run pytest -q tests/test_ollama.py::test_ask_retries_once_with_half_ctx_on_5xx
uv run ruff format . && uv run ruff check .
uv run mypy                              # strict; package configured in pyproject
uv run council-mcp                       # MCP server over stdio (wired via .mcp.json)
```

On this machine `uv` is not on PATH: in Git Bash prefix with `export PATH="$APPDATA/Python/Python312/Scripts:$PATH"`. Live smoke test of the local adapter needs `ollama serve` running and `COUNCIL_OLLAMA_URL=http://localhost:11434`.

Gates (same commands, run in a task worktree before review and after merge) are defined in `.council/council.json` → `gates`; output goes to `reports/<id>/gates.json`.

## Architecture (see design doc §2–5)

- **Plugin layer** (`.claude-plugin/plugin.json`, `commands/*.md` (flat — a subfolder would become a second namespace, `/council:council:x`), `agents/*.md`, `hooks/hooks.json`, `templates/`): slash commands `/council:plan run status answer review merge stop`, subagents `council-planner`, `council-reviewer`, `council-integrator`, a `UserPromptSubmit` hook that injects a summary of new `events.jsonl` entries. Fallback if plugin format differs: same files under `.claude/` installed by `council init`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sitkowsp/super-claude-code](https://github.com/sitkowsp/super-claude-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
