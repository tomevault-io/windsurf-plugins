---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
pip install -e ".[dev]"              # Install in dev mode (Claude + Codex bundled)
pytest                               # Run all tests
pytest tests/test_config.py -v       # Run a single test file
pytest -k "test_yaml_roundtrip"      # Run a specific test
ruff check src/ tests/               # Lint
ruff check --fix src/ tests/         # Lint with auto-fix
```

## Architecture

Codeband orchestrates multiple AI coding agents in parallel via the [Band.ai](https://band.ai) platform.

### Two agent patterns

1. **ClaudeSDK/Codex adapters** — Every agent that uses an LLM (Planner, Conductor, Coder, Plan Reviewer, Code Reviewer, Mergemaster) wraps `ClaudeSDKAdapter` or `CodexAdapter` from `band-sdk` (imported from the `band` module namespace, e.g. `from band.adapters import ClaudeSDKAdapter`). band-sdk 1.0.0 renamed the old `thenvoi` SDK namespace to `band`; the separate `thenvoi_rest` REST-client package is unrelated and still imported as `thenvoi_rest`. They expose an `.adapter` property for `Agent.create()`. Every role supports both Claude and Codex frameworks. Permissions are role-specific: Coders, Mergemaster, and Code Reviewer run with full access (`bypassPermissions` for Claude, `danger-full-access` for Codex) — Coders and Mergemaster in git worktrees, the Code Reviewer in an isolated scratch directory. Coordination-only agents (Plan Reviewer, Claude Conductor) use `permission_mode="dontAsk"` with `approval_mode=None` — the Claude CLI honors the worktree's `.claude/settings.json` allow list (see profiles in `workspace/init.py`) and deterministically denies everything else. The Codex Conductor gets the closest available equivalent via `sandbox="read-only"` + `approval_policy="never"` + an isolated temporary `cwd` outside the repo. That keeps native Codex file tools away from project files, but it is still weaker than Claude's true "MCP tools only" shape because the current Thenvoi Codex adapter does not expose a native-tool allowlist. Prompts are loaded via the shared `agents/prompts.py:load_prompt()` utility and are framework-portable — the `band_*` MCP tool names are injected by both adapters. See `player_claude.py` / `player_codex.py` as the canonical coding pair and `conductor.py` as the coordination example.

2. **Deterministic daemon** — The Watchdog runs as a plain `asyncio` task (not a Band.ai Agent). It polls the Band.ai REST API on an interval, applies deterministic threshold rules to detect stale agents, nudges them, and escalates to the Conductor on a second threshold crossing. No LLM. See `watchdog.py:WatchdogDaemon`.

All LLM calls — including the `cb prs --smart` / `cb issues --smart` CLI helpers (`utility_llm.py:one_shot_text`) — go through Claude Code SDK or Codex CLI, so a user can authenticate once via `ANTHROPIC_API_KEY`, `CLAUDE_CODE_OAUTH_TOKEN`, or `OPENAI_API_KEY` and the whole system works.

### Auth policy (API-key-first for both)

- **Claude is API-key-first, subscription by explicit opt-in.** Controlled by `claude.auth_mode` in `codeband.yaml` (`config.py:ClaudeConfig`, default `api_key`). In `api_key` mode `cli.py:_resolve_claude_auth` is a no-op — the Claude CLI uses `ANTHROPIC_API_KEY` natively (Anthropic Commercial Terms permit automated/parallel use). Only in `auth_mode: subscription` does it strip `ANTHROPIC_API_KEY` when an OAuth source exists (`CLAUDE_CODE_OAUTH_TOKEN`, macOS Keychain `security -s "Claude Code-credentials"`, or `$CLAUDE_CONFIG_DIR/.credentials.json`), keeping it as a usage-limit fallback. The mode is read at CLI entry by `_claude_auth_mode()` via a defensive direct YAML read (config isn't fully loaded yet). Rationale: Anthropic's Consumer Terms restrict automated subscription use, so the compliant default is the API key — see `docs/AUTHENTICATION.md`.
- **Codex is API-key-first.** When both `OPENAI_API_KEY` and a `~/.codex/auth.json` ChatGPT session exist, the API key wins (per OpenAI's own automation guidance and `docker/entrypoint.sh`). Parallel Codex workers exhaust subscription quotas quickly.
- **Preflight in `cb run`** (`preflight.py`) makes one tiny call through each CLI before spawning agents. It pattern-matches known error text ("Credit balance is too low", "usage limit reached", "rate_limit_error", "please log in", etc.) and fails fast with a remediation hint. In `api_key` mode it also fails fast (no API call) when `ANTHROPIC_API_KEY` is unset, so the subscription path is never taken implicitly. Without this, auth/billing errors surface as plain assistant text posted into chat rooms — the watchdog stays happy and the whole swarm idles silently. `--skip-preflight` bypasses for CI/offline. The Codex preflight only runs when the config has at least one Codex agent.
- **Doctor** (`doctor.check_claude_auth`) reads `claude.auth_mode`: in `api_key` mode it FAILs when no `ANTHROPIC_API_KEY`; in `subscription` mode it WARNs that Consumer Terms restrict automated subscription use and points back to the API-key path.

### Worker pool architecture


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [band-ai/codeband](https://github.com/band-ai/codeband) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
