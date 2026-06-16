---
trigger: always_on
description: Generates nginx-strict.conf / nginx-open.conf /
---

# Last Light — Development Guide

> **Architectural reference:** `spec/README.md` is the rebuild-grade
> specification — twelve pages covering every layer with schemas,
> invariants, and rebuild notes. Use this CLAUDE.md for day-to-day
> orientation; use `spec/` when you need the contract.
>
> **OpenCode → agentic-pi:** parts of this file still reference OpenCode
> (the original runtime). The current runtime is `agentic-pi` for
> sandboxed phases and `pi-ai` for in-process chat. The spec reflects
> the current state; this guide will be updated in a follow-up pass.

A GitHub repository maintenance agent. It listens for events (GitHub webhooks
and Slack messages), classifies them, and runs an AI agent against a target
repo via the **OpenCode** runtime (`sst/opencode`). Everything non-trivial —
triage, PR review, the full Architect→Executor→Reviewer build cycle, health
reports — is expressed as a **YAML workflow** the harness executes
phase-by-phase.

## Runtime

OpenCode is provider-agnostic. The harness defaults to
`openai/gpt-5.5` and accepts any `provider/model` string OpenCode
supports (`anthropic/…`, `openai/…`, `openrouter/<vendor>/<model>`, etc.).
API credentials are read from `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`,
and/or `OPENROUTER_API_KEY` on the harness env; set whichever provider(s)
match your `OPENCODE_MODEL` / `OPENCODE_MODELS`. No `claude` CLI, no
Anthropic SDK in the runtime path.

The cheap-helper path (`src/engine/llm.ts`, used by screener + classifier)
bypasses OpenCode and dispatches directly to the same three providers.
`defaultFastModel()` prefers Anthropic > OpenAI > OpenRouter when multiple
keys are set — direct provider routes avoid OpenRouter's per-token markup
when possible.

Two execution surfaces:
- **Sandbox** — `opencode run --format json` invoked per workflow phase
  inside a Docker container (`src/sandbox/docker.ts`). Stream parsed to
  capture session id, tokens, cost, stop reason. Used by every YAML
  workflow.
- **`opencode serve` (chat)** — one long-lived HTTP server on harness
  boot. Each messaging thread maps to one OpenCode session; `POST
  /session/{id}/message` per turn. Replaces the in-process Agent SDK
  query() the chat path used pre-fork.

Both surfaces write a Claude-SDK-style envelope jsonl to
`$STATE_DIR/opencode-home/projects/<slug>/<sessionId>.jsonl` (the
"shim") so the dashboard's `SessionReader` keeps working unchanged. The
shim is `src/engine/opencode-shim.ts`.

## Repo layout

```
src/
  index.ts              Main entry — wires connectors, boots opencode
                        serve, starts the cron scheduler and admin dashboard.
  config.ts             Layered config load: config/default.yaml +
                        optional $LASTLIGHT_OVERLAY_DIR/config.yaml + env
                        overrides. Secrets stay env-only. Exposes
                        getRuntimeConfig / getManagedRepos / getRoutes /
                        getPublicConfig.
  cli.ts                Thin client that POSTs to a running server.
  connectors/           Platform abstraction — every event source emits an
                        EventEnvelope so the engine never sees raw payloads.
    github-webhook.ts   GitHub App webhook → EventEnvelope.
    slack/              Slack Socket Mode + mrkdwn formatter.
    messaging/          Base class for all messaging platforms
                        (slack now, discord later). Owns SessionManager — the
                        per-thread conversation store.
  engine/
    router.ts           Deterministic, code-based routing of EventEnvelope
                        → { skill, context }. Classifies build intent via a
                        small LLM call. No LLM decides the tab.
    opencode-executor.ts  Runs one agent session via opencode run inside a
                        Docker sandbox. Parses --format json stream for
                        tokens / cost / session id / stop reason and feeds
                        the dashboard shim.
    opencode-chat-server.ts  Supervisor + typed HTTP client for the
                        long-lived opencode serve process. Per-session
                        in-flight chain serializes same-sessionId calls
                        (e.g. two messages in one Slack thread) while
                        keeping cross-session traffic parallel.
    chat.ts             Chat skill — creates/resumes an OpenCode session
                        per Slack thread, posts the turn, writes the
                        dashboard envelope jsonl, returns ChatResult
                        metrics for the executions row.
    opencode-shim.ts    ClaudeJsonlShim: translates OpenCode events
                        (text / tool_use / error) into Claude-SDK
                        envelope jsonl lines under opencode-home/projects/.
                        MCP tool name shim (github_<tool> → mcp_github_<tool>
                        for the dashboard tool-family classifier).
    profiles.ts         ExecutorConfig / ExecutionResult / GitSandboxAccess
                        types + GITHUB_PERMISSION_PROFILES + loadAgentContext.
                        Imported by runner.ts, chat.ts, opencode-executor.ts.
    llm.ts              One-shot LLM helper for screen.ts + classifier.ts —

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cliftonc/lastlight](https://github.com/cliftonc/lastlight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
