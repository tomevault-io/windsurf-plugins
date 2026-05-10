---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development Commands

This project uses [just](https://github.com/casey/just) as the user-facing command runner, [Task](https://taskfile.dev) (`Taskfile.yml`) underneath for per-target fingerprint caching, and [mise](https://mise.jdx.dev) for tool versioning (Go 1.26.2). `just` recipes for `build`, `test`, `lint`, `vet`, `fmt-check`, `lint-ui`, `build-ui`, `test-ui`, `openapi`, and `hook` delegate to `mise x -- task <name>`; Task skips any step whose declared sources/inputs haven't changed. Cache lives in `.task/` (gitignored). Bust a single step with `mise x -- task <name> --force`; bust the whole hook chain with `JUST_HOOK_FORCE=1 just hook`.

```bash
just build                    # Build binary → pkg/bin/denkeeper
just serve                    # Run via go run (accepts optional config path)
just test                     # All Go tests with -race
just test-v                   # Verbose test output
just test-pkg internal/agent  # Single package
just test-ui                  # Web UI tests (vitest)
just lint                     # golangci-lint
just fmt                      # gofmt -w .
just check                    # fmt-check + vet + lint + test + test-ui (CI equivalent)
just hook                     # Equivalent of `just check` with minimal output (for agent use)
just scan                     # Security scans (gosec + govulncheck)
just build-ui                 # Build web UI (auto-run by build/test/vet/lint when web/dist is missing)
just build-full               # Build web then binary
just openapi                  # Generate OpenAPI spec (requires swag CLI)
just web-dev                  # Vite dev server with hot-reload
just test-integration         # E2E integration tests
```

Note: if you need to run the full test suite, you should prefer `just hook` as that command is cached

## Architecture

Denkeeper is a single-binary personal AI agent with multi-agent routing. Messages flow through:

```
Adapter (Telegram/Discord) ─┐
Web Dashboard (WS/SSE) ─────┼→ Dispatcher → Engine (per agent) → LLM Router → Provider (Anthropic/OpenRouter/OpenAI/Ollama)
REST API (/api/v1/chat) ────┘                    ↕                    ↕
                                             MemoryStore          CostTracker
                                             (SQLite)              + Pricing Registry
```

**Dispatcher** (`internal/agent/dispatcher.go`) routes messages to the correct Engine based on channel bindings or legacy adapter bindings. Falls back to the `"default"` agent. Handles `tool_approval` ChatEvents by sending inline keyboard approval messages. When channels are configured, the dispatcher intercepts `/session` commands to allow runtime channel switching.

**Channels** (`internal/agent/channel.go`) are named routing endpoints that decouple sessions from the rigid 1:1 agent-adapter binding. A channel points to an agent and can be bound to multiple adapters (cross-adapter session sharing). Config: `[[channels]]` in TOML with `name`, `agent`, `adapters`. When `[[channels]]` is absent, channels are auto-synthesized from agent `adapters` bindings (backward compatible). Conversation ID format: `"chan:{channel_name}"`. Users switch channels via `/session <name>` in adapters; selections persist in SQLite (`active_channels` table) across restarts. Resolution priority: active override (/session) > specific binding > wildcard binding > legacy resolveAgent fallback.

**Engine** (`internal/agent/engine.go`) is the per-agent orchestrator. Pipeline: check permissions → get/create conversation → store user message → load history → build system prompt (persona + skills) → call `Router.Complete()` → tool-call loop (with supervised approval if applicable) → emit usage event → extract memory update → store assistant message → return text.

**Three key interfaces**:
- `adapter.Adapter` — platform integrations (Telegram, Discord)
- `llm.Provider` — LLM backends (Anthropic, OpenRouter, OpenAI, Ollama)
- `agent.MemoryStore` — conversation persistence (SQLite)

**Multi-agent config**: `[[agents]]` in TOML. Each agent has `name`, `persona_dir`, `adapters`, `llm_provider`, `llm_model`, `session_tier`. If no `[[agents]]` section exists, a single `"default"` agent is synthesized. `llm_provider` overrides the global `default_provider` for that agent, enabling different agents to use different LLM backends.

**Named provider instances**: `[[llm.providers]]` array allows multiple instances of the same provider type (e.g. two OpenAI-compatible endpoints). Each entry has `name`, `type` (`anthropic`/`openai`/`openrouter`/`ollama`), `api_key`, `base_url`, `organization`. Legacy `[llm.openai]` single-slot syntax is still supported and auto-converted. Per-agent `llm_provider` references instances by name.

**Data directory**: All default paths (db, persona, skills) are derived from a single base directory. Set via `DENKEEPER_DATA_DIR` env var, `data_dir` in TOML, or defaults to `~/.denkeeper`. The Helm chart sets `DENKEEPER_DATA_DIR=/data` so everything lands on the writable PVC.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Temikus/denkeeper](https://github.com/Temikus/denkeeper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
