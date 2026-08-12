---
trigger: always_on
description: Personal voice assistant with Pi Zero nodes and self-hosted microservices.
---

# Jarvis

Personal voice assistant with Pi Zero nodes and self-hosted microservices.

**See also:**
- [RULES.md](RULES.md) — development rules (working style, coding style, TDD, performance targets)
- **Each service has its own CLAUDE.md** — read the relevant one *first* when working in that service. This meta doc is for cross-cutting concerns only.

---

## Core Principles

1. **Fully private and open source** — no cloud dependencies by default, all data stays local
2. **Self-hostable with optional cloud** — same open-source codebase for both
3. **Fully extensible** — add capabilities by implementing `IJarvisCommand` (see `jarvis-command-sdk` for the interface, `jarvis-node-setup` for the runtime)

---

## For Claude: Use MCP Tools

**Prefer jarvis-mcp tools over direct curl/HTTP/docker calls** when available:

| Instead of... | Use MCP tool... |
|---|---|
| `curl localhost:7702/health` etc. | `debug_health` |
| Querying logs via curl | `query_logs`, `logs_tail`, `get_log_stats` |
| Getting service info | `debug_service_info` |
| `docker ps` | `docker_ps` |
| `docker logs <container>` | `docker_logs` |
| `docker restart <container>` | `docker_restart` / `docker_stop` / `docker_start` |
| `docker compose up/down` | `docker_compose_up` / `docker_compose_down` / `docker_compose_list` |

> jarvis-mcp is **potentially deprecated** — see its CLAUDE.md. The tools above still work today; don't extend the surface without checking first.

---

## Services (directory)

| Service | Port | One-liner |
|---|---|---|
| jarvis-config-service | 7700 | Service registry + first-boot bootstrap + settings gateway |
| jarvis-auth | 7701 | Users, JWT, app-to-app, nodes, households |
| jarvis-logs | 7702 | Loki/Grafana fronted by FastAPI |
| jarvis-command-center | 7703 | The brain — voice orchestration, memory, tools, routines |
| jarvis-llm-proxy-api | 7704 / 7705 | OpenAI-compatible API + model service + queue worker |
| jarvis-whisper-api | 7706 | STT (in-process pywhispercpp) + speaker recognition |
| jarvis-tts | 7707 | TTS (Piper / Kokoro) with streaming |
| jarvis-settings-server | 7708 | **Deprecation candidate** — use config-service's `/v1/settings/*` |
| jarvis-mcp | 7709 | **Potentially deprecated** — MCP server for Claude Code |
| jarvis-admin | 7710 | Web admin (Fastify backend + React SPA) |
| jarvis-notifications | 7712 | Push + inbox |
| jarvis-pantry | 7721 | Cloud package store + AI Forge |
| jarvis-web | 7722 | Browser chat (Next.js, rewrites to backends) |
| jarvis-recipes-server | 7030 | Recipes + meal planning |
| jarvis-ocr-service | 7031 | OCR (Tesseract / EasyOCR / Apple Vision) |

**Libraries:** `jarvis-log-client`, `jarvis-config-client`, `jarvis-web-scraper`, `jarvis-command-sdk`
**Clients:** `jarvis-node-setup` (Pi Zero), `jarvis-node-mobile`
**Shared infra:** PostgreSQL, Redis, MinIO, Mosquitto (MQTT)

Each service has its own CLAUDE.md with the dependency graph, recipes, invariants, and failure modes for that service. **Read it first.**

---

## Service communication patterns

| Auth mode | Header | Validated where |
|---|---|---|
| **User JWT** | `Authorization: Bearer <jwt>` | Locally in each service using shared `AUTH_SECRET_KEY` |
| **App-to-app** | `X-Jarvis-App-Id` + `X-Jarvis-App-Key` | Round-trip to `jarvis-auth /internal/app-ping` (or `/internal/validate-app`) |
| **Node** | `X-API-Key: node_id:node_key` | Round-trip to `jarvis-auth /internal/validate-node` (also checks per-service access) |
| **Admin** | `X-Jarvis-Admin-Token` (or service-specific `X-Admin-Token`) | Local env-var compare |

**Service discovery:** every service queries `jarvis-config-service /services` for URLs at startup (via `jarvis-config-client`, cached locally with 5min background refresh).

### Hot path: voice command from a node

```
Pi node ──(X-API-Key)──▶ jarvis-command-center ──┬─▶ jarvis-llm-proxy-api (inference)
                                                  ├─▶ jarvis-whisper-api (STT, via /media proxy)
                                                  ├─▶ jarvis-tts (audio out, streamed)
                                                  └─▶ jarvis-auth (node validation, speaker resolve)
```

Per-service detail lives in each service's CLAUDE.md.

---

## Cross-service environment variables

| Variable | Used by | Notes |
|---|---|---|
| `DATABASE_URL` | most services | PostgreSQL connection (each service has its own DB) |
| `AUTH_SECRET_KEY` | every service that validates JWTs | **Must match across all services.** In jarvis-auth it's `AUTH_SECRET_KEY`; older docs may say `SECRET_KEY` — that's stale. |
| `JARVIS_CONFIG_URL` | every service | Config-service URL (typically `http://localhost:7700`) |
| `JARVIS_APP_ID` / `JARVIS_APP_KEY` | every service | App-to-app credentials for outbound calls |
| `JARVIS_AUTH_ADMIN_TOKEN` | trusted infrastructure only | Master admin token for jarvis-auth `/admin/*` |
| `ADMIN_API_KEY` | command-center, notifications, pantry | Per-service admin endpoint protection (distinct from `JARVIS_AUTH_ADMIN_TOKEN`) |

Per-service env vars live in each service's CLAUDE.md.

---

## Development model (mixed local/Docker)

The `./jarvis` CLI handles platform differences automatically.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alexberardi/jarvis](https://github.com/alexberardi/jarvis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
