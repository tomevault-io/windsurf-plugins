---
trigger: always_on
description: TMA1 is local-first LLM agent observability, powered by GreptimeDB.
---


## What this repo is

TMA1 is local-first LLM agent observability, powered by GreptimeDB.
Three pillars (traces + metrics + logs) unified into wide events — all queryable via SQL.

- **Traces** are the core: GenAI / OpenClaw spans carry model, tokens, latency, status
- **Metrics** are derived from traces via Flow engine (no double-writing), and native OTel metrics are also accepted (auto-creates tables)
- **Logs** carry conversation content (conversation replay)
- **Cross-signal JOIN**: `trace_id` connects spans to conversations

The name comes from TMA-1 (Tycho Magnetic Anomaly-1) in *2001: A Space Odyssey*:
the monolith buried on the moon, silently recording everything until you dig it out.

Tagline: *"Your agent runs. TMA1 remembers."*

## High-level modules

| Path | Role |
|------|------|
| `server/` | Go binary: GreptimeDB process manager + HTTP server + dashboard |
| `server/cmd/tma1-server/` | Entry point + embedded FS mount |
| `server/internal/config/` | Env var config loading |
| `server/internal/install/` | Download and verify GreptimeDB binary |
| `server/internal/greptimedb/` | Start, stop, health-check GreptimeDB process + Flow init |
| `server/internal/handler/` | HTTP handlers: /health, /status, /api/query, /api/evaluate, /api/settings, /api/hooks, /api/hooks/stream (SSE), /v1/otlp/*, dashboard UI |
| `server/internal/hooks/` | Hook script installer for Claude Code integration |
| `server/internal/transcript/` | JSONL transcript watcher (Claude Code) + Codex / OpenClaw / Copilot CLI session log parsers |
| `server/web/` | Embedded dashboard (HTML + JS + CSS via embed.FS), 7 views: Claude Code, Codex, Copilot CLI, OpenClaw, OTel GenAI, Sessions, Prompts + Agent Canvas |
| `site/` | Astro landing page → GitHub Pages → tma1.ai |
| `.claude-plugin/` | Claude Code Marketplace registration |
| `claude-plugin/` | Claude Code plugin: skills for setup + inline queries |
| `clawhub-skill/tma1/` | SKILL.md — ClawHub-format skill (OpenClaw integration) |

## Architecture

```
Agent (Claude Code / Codex / Copilot CLI / OpenClaw / any GenAI app)
    │  OTLP/HTTP → http://localhost:14318/v1/otlp
    │  Hook events → http://localhost:14318/api/hooks (Claude Code)
    │  JSONL transcripts → ~/.claude/projects/ (CC) / ~/.codex/sessions/ (Codex) /
    │                      ~/.copilot/session-state/ (Copilot CLI) / ~/.openclaw/agents/ (OpenClaw)
    ▼
tma1-server  port 14318
    │  reverse-proxies OTLP to GreptimeDB
    │  auto-injects x-greptime-pipeline-name for trace requests
    │  receives hook events → tma1_hook_events + SSE broadcast
    │  watches JSONL transcripts → tma1_messages
    ▼
GreptimeDB  (managed by tma1-server)
    │  Flow engine → tma1_*_1m aggregation tables
    │  HTTP SQL API  port 14000
    ▼
Browser dashboard (served by tma1-server)
    ├── Claude Code view: Overview, Tools, Cost, Anomalies, Traces, Sessions→ (from OTel metrics + logs + traces)
    ├── Codex view: Overview, Tools, Cost, Anomalies, Sessions→ (from OTel logs with scope_name codex_*)
    ├── Copilot CLI view: Overview, Tools, Cost, Sessions→ (from ~/.copilot/session-state/*/events.jsonl)
    ├── OpenClaw view: Overview, Traces, Cost, Search (from openclaw.* trace attrs)
    ├── OTel GenAI view: Overview, Traces, Cost, Security, Search (from gen_ai.* trace attrs)
    ├── Sessions view: Session list, full-screen detail overlay (two-column: Insights + Timeline), file heatmap, agent hierarchy, waterfall, canvas animation
    │   ├── CC/Codex/Copilot CLI "Sessions→" is a link that jumps to Sessions view with agent_source filter
    │   ├── Replay mode: replay past sessions as agent orchestration animation
    │   └── Live mode: real-time SSE streaming of hook events → canvas visualization
    └── Prompts view: Prompt evaluation & improvement (heuristic scoring + optional LLM-as-judge)
        ├── Overview: score distribution, trend, top suggestions, dimension breakdown
        ├── Prompts: card-based list with per-prompt scoring, suggestions, optional LLM deep eval
        └── Patterns: verb-based grouping (fix/add/implement/debug/...) with avg score/cost/turns
```

OTel data goes through tma1-server's OTLP proxy (`/v1/otlp/*`), which forwards to GreptimeDB (port 14000) and auto-injects the `x-greptime-pipeline-name: greptime_trace_v1` header for trace requests. Agents should send OTLP to `http://localhost:14318/v1/otlp`.

Hook events from Claude Code arrive via `POST /api/hooks` (configured as command hooks in `~/.claude/settings.json`, using the auto-installed hook script at `~/.tma1/hooks/tma1-hook.sh` on Unix/macOS or `%USERPROFILE%\.tma1\hooks\tma1-hook.ps1` on Windows). Claude Code's HTTP hook type requires HTTPS, so command hooks with curl are used instead. Codex session logs are auto-discovered from `~/.codex/sessions/` without any hook configuration. Copilot CLI session logs are auto-discovered from `~/.copilot/session-state/` without any hook configuration.

## Data sources

Six data paths, depending on the agent:

**Claude Code** → OTel metrics + logs + traces + hooks + JSONL transcripts:

| Table | Type | Content |
|-------|------|---------|
| `claude_code_token_usage_tokens_total` | Metric (counter) | Tokens by model + type (input/output/cacheRead/cacheCreation) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tma1-ai/tma1](https://github.com/tma1-ai/tma1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
