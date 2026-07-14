---
trigger: always_on
description: > Instructions for AI coding agents (Claude Code, OpenCode, Copilot, Cursor, Codex, Antigravity) operating in this repo.
---

# AGENTS.md

> Instructions for AI coding agents (Claude Code, OpenCode, Copilot, Cursor, Codex, Antigravity) operating in this repo.
>
> **Behavioural SSOT lives in dotfiles:** `$DOTFILES_REPO_DIR/AGENTS.md` (the dotfiles repo root, resolved per-machine via the path cascade: explicit env → `~/.config/dotfiles/machine.json` → `env-contract.json` default; ADR-025). Read it FIRST — Identity, Standing Orders, Decision Hierarchy, Model Selection, Neural Hive protocol, MCP usage rules, Spec-Driven Development gate, and operational rules. This file adds ONLY what is specific to the Hive repo. The Claude-specific tooling overlay is `.claude/CLAUDE.md`.

## What this repo is

> **Hive** — vault-native AI orchestration: a unified MCP server giving AI coding assistants on-demand access to an Obsidian vault plus delegation to local/cheap workers.

Build/operate docs live in [`docs/`](docs/) (docs-as-code): [`docs/adr/`](docs/adr/) (architecture decisions + `sequence-diagrams.md`), [`docs/runbooks/`](docs/runbooks/), [`docs/troubleshooting/`](docs/troubleshooting/), [`docs/lessons.md`](docs/lessons.md). Per-feature specs live in `specs/`. Task state lives in the **bitácora** GitHub Project (filter by `Repo` = hive), not here. The cross-project brain and AI memory live in the maintainer's vault.

## Big-picture architecture

Hive is an MCP server (stdio transport, FastMCP framework) with three responsibilities:

1. **Vault tools** — query, search, list, write, patch markdown files in an Obsidian vault. All writes auto-commit to git (best-effort; git failure never crashes the server) and validate YAML frontmatter.
2. **Session tools** — `session_briefing` assembles tasks + lessons + git log + health in one call so an AI client gets ~50 lines of context instead of ~800.
3. **Worker tools** — `delegate_task` and `capture_lesson` route work down a cost ladder: Ollama (local, free) → OpenRouter free tier → OpenRouter paid (gated by $1/mo SQLite budget) → reject.

The package layout follows a deliberate split: `server.py` is a thin registration layer only. Each `_vault_*.py` / `_workers.py` module owns one tool family and registers its tools onto the FastMCP instance via a `register_*(mcp, ctx)` function. State lives in `ServerContext` (a dataclass in `_context.py`) and is passed to every handler — there is no module-level mutable state.

| Path | Role |
|---|---|
| `src/hive/server.py` | Thin registration layer — `create_server()`, resources, prompts |
| `src/hive/_context.py` | `ServerContext` dataclass — shared state for all handlers |
| `src/hive/_helpers.py` | Pure helpers — path resolution, formatting, git ops, tracking |
| `src/hive/_vault_read.py` | `vault_list`, `vault_query`, `vault_search`, `session_briefing` |
| `src/hive/_vault_write.py` | `vault_write`, `vault_patch` (both auto-commit to git) |
| `src/hive/_vault_health.py` | `vault_health` + health report builder |
| `src/hive/_workers.py` | `capture_lesson`, `delegate_task`, `worker_status` |
| `src/hive/_compat.py` | MCP cancellation shim — see "Compat shim" below |
| `src/hive/config.py` | `HiveSettings` (pydantic-settings, `HIVE_*` env vars) |
| `src/hive/budget.py` | SQLite budget tracker ($1/mo default cap, WAL mode) |
| `src/hive/clients.py` | Async HTTP clients (Ollama + OpenRouter, httpx) |
| `src/hive/relevance.py` | EMA-based section relevance scoring |
| `src/hive/frontmatter.py` | YAML frontmatter parse/validate/generate |
| `site/` | Astro + Starlight bilingual (EN/ES) docs site |

### Compat shim (do not delete blindly)

`src/hive/_compat.py` monkey-patches `mcp.shared.session.RequestResponder.__exit__` to swallow the spurious `CancelledError` that anyio re-raises after a cancelled tool call has already responded. Without it, a client sending `notifications/cancelled` kills the stdio receive loop and every subsequent call hangs (hive issue #75). The patch fires only on the exact failure mode and degrades silently if upstream removes the symbol. Delete only after confirming the upstream MCP fix has shipped.

**Upstream tracker:** [modelcontextprotocol/python-sdk#2610](https://github.com/modelcontextprotocol/python-sdk/issues/2610). `mcp` pinned `>=1.26,<2.0` in `pyproject.toml` so a major `RequestResponder` refactor cannot silently break the shim. **Escalation deadline:** 2026-06-12 — if upstream is still silent, port the fix upstream ([#127](https://github.com/mlorentedev/hive/issues/127)).

### Worker routing order

`delegate_task` tries clients in this order, falling through on failure or unavailability:

1. **Ollama** `qwen2.5-coder:7b` (local) — free, primary
2. **OpenRouter** `qwen/qwen3-coder:free` — free tier fallback
3. **OpenRouter** paid (`qwen/qwen3-coder`) — only if caller passes `max_cost_per_request > 0` AND `BudgetTracker` allows it
4. **Reject** — surface the error to the client; Claude handles fallback

## MCP tool schema rules (load-bearing)

These rules are not stylistic — violating them breaks the server in subtle, hard-to-diagnose ways.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mlorentedev/hive](https://github.com/mlorentedev/hive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
