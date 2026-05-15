---
trigger: always_on
description: > Compact AI operating system built in Elixir/OTP. Personal AI assistant gateway
---

# CLAUDE.md — Traitee

> Compact AI operating system built in Elixir/OTP. Personal AI assistant gateway
> with hierarchical memory, cognitive security, and multi-channel support.

## Quick Reference

```bash
mix setup              # Install deps + create DB + migrate
mix test               # Run unit tests (auto-migrates)
mix lint               # Format check + Credo strict
mix quality.ci         # Format + Credo + Dialyzer
mix format             # Auto-format code
mix credo --strict     # Static analysis
mix dialyzer           # Type checking (slow first run — builds PLTs)
```

## Tech Stack

- **Elixir ~> 1.17** on **OTP 27** (BEAM VM)
- **Phoenix ~> 1.7** with **Bandit** HTTP server (port 4000)
- **SQLite** via `ecto_sqlite3` — single file at `~/.traitee/traitee.db`
- **Nx** for vector similarity (cosine distance in ETS)
- **Nostrum** (Discord), **ExGram** (Telegram) for channel integrations
- **Req** for HTTP (LLM API calls)
- **Mox** + **ExUnit** for testing, **ExCoveralls** for coverage

## Project Layout

```
lib/
  traitee/
    application.ex         OTP supervision tree (19 children, 9 ETS table inits)
    config.ex              Multi-source TOML config loader (:persistent_term)
    router.ex              Inbound message routing (security + pairing + commands)
    session.ex             Session facade (Registry + DynamicSupervisor)
    workspace.ex           Workspace file management (SOUL/AGENTS/TOOLS/BOOT.md)
    activity_log.ex        Non-blocking ETS activity logger (500 entries/session)
    doctor.ex              System diagnostics (11 health checks)
    session/               GenServer per user, lifecycle, inter-session
    memory/                3-tier: STM (ETS) → MTM (summaries) → LTM (knowledge graph)
                           + vector (Nx cosine), hybrid search, MMR, temporal decay,
                           query expansion, batch embedder, compactor
    context/               Token-aware prompt assembly with budget allocation + continuity
    llm/                   Provider abstraction: OpenAI, Anthropic, xAI, Ollama
    security/              16 modules: 8-layer cognitive pipeline + 4-layer filesystem pipeline
                           (sanitizer, judge, threat_tracker, cognitive, canary, system_auth,
                            output_guard, io_guard, sandbox, filesystem, exec_gate, docker,
                            audit, pairing, allowlist, rate_limiter)
    tools/                 12 built-in tools (bash, file, browser, web_search, memory,
                           sessions, cron, channel_send, skill_manage, workspace_edit,
                           delegate_task, task_tracker) + dynamic runtime tools
    channels/              Discord, Telegram, WhatsApp, Signal, streaming, typing
    hooks/                 9 hook points with chainable handlers (12 built-in hooks)
    skills/                Loader (3-tier progressive disclosure) + registry (60s rescan)
    routing/               Multi-agent router with 5-tier priority + bindings
    cron/                  Scheduler with cron expression parser
    process/               Cross-platform executor + concurrency lanes
    auto_reply/            Debouncer + command registry (20+ commands) + pipeline
    browser/               Playwright bridge (Node.js JSON-RPC subprocess)
    daemon/                OS service management (Windows/Linux/macOS)
    secrets/               Credential store + manager
    media/                 Pipeline + text extractor
    delegation/            Parallel subagent orchestration (max 5, max 25 tool iters)
    cli/                   Terminal display utilities
    onboard/               Interactive 12-step setup wizard
  traitee_web/
    controllers/           health, webhook, openai_proxy (OpenAI-compatible API)
    channels/              Phoenix WebSocket (chat_channel, user_socket)
  mix/tasks/               10 CLI tasks: chat, serve, send, doctor, memory, cron, daemon, onboard, pairing, security
config/                    config.exs, dev.exs, test.exs, prod.exs, runtime.exs
priv/repo/migrations/      SQLite migrations (sessions, messages, summaries, entities, relations, facts, cron_jobs)
priv/browser/              Node.js Playwright bridge (bridge.js, 14 actions, multi-tab)
test/                      ~41 test files mirroring lib/ structure
```

## Architecture

### Supervision Tree

Single `one_for_one` supervisor starts all services. 9 ETS tables are initialized
**before** the supervision tree to guarantee lock-free reads from boot:

```
ETS tables (pre-boot):
  Tools.Registry, Security.RateLimiter, Security.ThreatTracker,
  Security.Canary, Security.Filesystem, Memory.Vector,
  Tools.TaskTracker, ActivityLog, Security.SystemAuth

Traitee.Application (19 children)
├── Repo (SQLite/Ecto)
├── PubSub (Phoenix — config changes, webchat)
├── Hooks.Engine (GenServer — 9 hook points, 12 built-in handlers)
├── Config.HotReload (GenServer — 5s file poll, PubSub broadcast)
├── LLM.Router (GenServer — failover + usage tracking)
├── Memory.Compactor (GenServer — async STM→MTM→LTM)
├── Memory.BatchEmbedder (GenServer — batches of 20, 5s tick)
├── Skills.Registry (GenServer — 60s rescan, :persistent_term cache)
├── Security.Audit (GenServer — ETS ring buffer 10K events)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [blueberryvertigo/traitee](https://github.com/blueberryvertigo/traitee) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
