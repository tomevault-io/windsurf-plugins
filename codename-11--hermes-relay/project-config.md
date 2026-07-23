---
trigger: always_on
description: > Read [AGENTS.md](AGENTS.md) first. It is the provider-neutral canonical agent
---

# Hermes-Relay — Claude Code Adapter

> Read [AGENTS.md](AGENTS.md) first. It is the provider-neutral canonical agent
> context. Branch, release, staging, and hotfix rules live in `AGENTS.md` and
> [RELEASE.md](RELEASE.md); this file only adds Claude-specific project and tool
> guidance. Then read `docs/spec.md` and `docs/decisions.md`.

## What This Is

A native Android app (Kotlin + Jetpack Compose) paired with an optional Python relay plugin/server (aiohttp) for the Hermes agent platform. Vanilla Hermes chat, Manage, and dashboard voice work against unmodified upstream Hermes. The Relay plugin adds phone control, terminal, remote desktop tooling, extra voice engines, and dashboard Relay management via the official Hermes web dashboard.

**Current state:** Reference latest released version for stable state and current dev branch for working state. The default no-plugin path supports chat, Manage, and voice on vanilla upstream Hermes. Chat auto-prefers the dashboard `/api/ws` gateway transport when Manage auth is ready, then falls back to API-server SSE routes. Vanilla Hermes voice uses dashboard `/api/audio/*` with the Manage session. Relay remains an additive power path for terminal, bridge/device control, notification companion, extra/provider-native voice, remote access, and desktop tooling. Two Android product flavors ship: `googlePlay` (conservative, no unattended Device Control surface) and `sideload` (full-capability).

## Architecture

```
Phone (WS)       -> Hermes dashboard (:9119)     [vanilla Hermes gateway chat, live thinking]
Phone (HTTP/SSE) -> Hermes API Server (:8642)    [vanilla Hermes chat fallback, sessions, runs]
Phone (HTTP)     -> Hermes dashboard (:9119)     [vanilla Hermes Manage + voice]
Phone (WSS/HTTP) -> Relay plugin/server (:8767)  [optional bridge, terminal, relay voice, remote tools]
```

The Vanilla Hermes path must stay upstream-only. API-server bearer auth and dashboard cookie auth are separate. Terminal and bridge require Relay pairing; Vanilla Hermes chat, Manage, and dashboard voice must not.

### Upstream Hermes API Reference

**IMPORTANT:** Always verify endpoints against the actual hermes-agent source (`gateway/platforms/api_server.py`). The upstream repo is the source of truth — not our docs, not our memory, not assumptions from other frontends.

**Vanilla Hermes endpoints (confirmed in hermes-agent source):**


| Endpoint                                | Purpose                                                  | Tool Call Format                                                                                                               |
| --------------------------------------- | -------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `POST /v1/chat/completions`             | OpenAI-compatible chat (stream=true for SSE)             | Inline markdown text (``💻 terminal``) — no separate tool events                                                               |
| `POST /v1/runs`                         | Start an agent run                                       | Returns `run_id`                                                                                                               |
| `GET /v1/runs/{run_id}/events`          | SSE stream of run lifecycle events                       | **Structured events**: `tool.started`, `tool.completed`, `message.delta`, `reasoning.available`, `run.completed`, `run.failed` |
| `POST /v1/responses`                    | OpenAI Responses API format                              | Structured `function_call` objects (non-streaming only)                                                                        |
| `GET /v1/capabilities`                  | Machine-readable feature + endpoint discovery            | Use before assuming optional surfaces exist                                                                                    |
| `GET /v1/models`                        | List available models                                    | —                                                                                                                              |
| `GET /v1/skills`                        | Read-only skill list for the API-server agent            | `{"object":"list","data":[...]}`                                                                                               |
| `GET /v1/toolsets`                      | Read-only API-server toolset inventory                   | `{"object":"list","platform":"api_server","data":[...]}`                                                                       |
| `GET/POST/PATCH/DELETE /api/sessions/*` | Native session CRUD, messages, fork, sync chat, SSE chat | Upstream merged via NousResearch/hermes-agent PR #33134                                                                        |
| `GET /health`                           | Health check                                             | —                                                                                                                              |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Codename-11/hermes-relay](https://github.com/Codename-11/hermes-relay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
