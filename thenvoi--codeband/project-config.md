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

1. **ClaudeSDK/Codex adapters** — Every agent that uses an LLM (Planner, Conductor, Coder, Plan Reviewer, Code Reviewer, Mergemaster) wraps `ClaudeSDKAdapter` or `CodexAdapter` from `band-sdk` (Python imports still use the `thenvoi` module namespace — the PyPI package was renamed but module paths stayed). They expose an `.adapter` property for `Agent.create()`. Every role supports both Claude and Codex frameworks. Permissions are role-specific: Coders, Mergemaster, and Code Reviewer run with full access (`bypassPermissions` for Claude, `danger-full-access` for Codex) — Coders and Mergemaster in git worktrees, the Code Reviewer in an isolated scratch directory. Coordination-only agents (Plan Reviewer, Claude Conductor) use `permission_mode="dontAsk"` with `approval_mode=None` — the Claude CLI honors the worktree's `.claude/settings.json` allow list (see profiles in `workspace/init.py`) and deterministically denies everything else. The Codex Conductor gets the closest available equivalent via `sandbox="read-only"` + `approval_policy="never"` + an isolated temporary `cwd` outside the repo. That keeps native Codex file tools away from project files, but it is still weaker than Claude's true "MCP tools only" shape because the current Thenvoi Codex adapter does not expose a native-tool allowlist. Prompts are loaded via the shared `agents/prompts.py:load_prompt()` utility and are framework-portable — the `thenvoi_*` MCP tool names are injected by both adapters. See `player_claude.py` / `player_codex.py` as the canonical coding pair and `conductor.py` as the coordination example.

2. **Deterministic daemon** — The Watchdog runs as a plain `asyncio` task (not a Band.ai Agent). It polls the Band.ai REST API on an interval, applies deterministic threshold rules to detect stale agents, nudges them, and escalates to the Conductor on a second threshold crossing. No LLM. See `watchdog.py:WatchdogDaemon`.

All LLM calls — including the `cb prs --smart` / `cb issues --smart` CLI helpers (`utility_llm.py:one_shot_text`) — go through Claude Code SDK or Codex CLI, so a user can authenticate once via `ANTHROPIC_API_KEY`, `CLAUDE_CODE_OAUTH_TOKEN`, or `OPENAI_API_KEY` and the whole system works.

### Auth policy (asymmetric by design)

- **Claude is subscription-first.** `cli.py:_resolve_claude_auth` strips `ANTHROPIC_API_KEY` at startup whenever any OAuth source exists: `CLAUDE_CODE_OAUTH_TOKEN`, macOS Keychain (`security -s "Claude Code-credentials"`), or `$CLAUDE_CONFIG_DIR/.credentials.json` (default `~/.claude/.credentials.json`). This is because the Claude CLI's own precedence puts API key *above* OAuth — without intervention, a user with `claude` logged in locally would silently pay per token.
- **Codex is API-key-first.** When both `OPENAI_API_KEY` and a `~/.codex/auth.json` ChatGPT session exist, the API key wins (per OpenAI's own automation guidance and `docker/entrypoint.sh`). Parallel Codex workers exhaust subscription quotas quickly.
- **Preflight in `cb run`** (`preflight.py`) makes one tiny call through each CLI before spawning agents. It pattern-matches known error text ("Credit balance is too low", "usage limit reached", "rate_limit_error", "please log in", etc.) and fails fast with a remediation hint. Without this, auth/billing errors surface as plain assistant text posted into chat rooms — the watchdog stays happy and the whole swarm idles silently. `--skip-preflight` bypasses for CI/offline. The Codex preflight only runs when the config has at least one Codex agent.
- **Doctor** (`doctor.check_claude_auth`) warns when `ANTHROPIC_API_KEY` is set alongside host subscription OAuth — Codeband auto-prefers at run-time, but the WARN flags the suboptimal `.env` for cleanup.

### Worker pool architecture

Coders, Code Reviewers, Planners, and Plan Reviewers are **pool roles** — each is declared in `codeband.yaml` as `{framework: {count, model, description?}}` under `agents.{coders, reviewers, planners, plan_reviewers}`. Conductor and Mergemaster are **singletons**. Pool member identities follow `{role}-{framework}-{index}` (e.g., `coder-claude_sdk-0`); Band.ai display names are title-cased (`Coder-Claude-0`). The default `cb init` config is 8 agents total — fits Band.ai's free-tier 10-agent cap.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thenvoi/codeband](https://github.com/thenvoi/codeband) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
