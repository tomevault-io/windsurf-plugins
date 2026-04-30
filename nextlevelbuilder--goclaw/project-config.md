---
trigger: always_on
description: PostgreSQL multi-tenant AI agent gateway with WebSocket RPC + HTTP API.
---

# GoClaw Gateway

PostgreSQL multi-tenant AI agent gateway with WebSocket RPC + HTTP API.

## Language

Always respond in the same language as the user's prompt. If the user writes in Vietnamese, respond in Vietnamese. If in English, respond in English. Match the user's language naturally.

## Tech Stack

**Backend:** Go 1.26, Cobra CLI, gorilla/websocket, pgx/v5 (database/sql, no ORM), golang-migrate, go-rod/rod, telego (Telegram)
**Web UI:** React 19, Vite 6, TypeScript, Tailwind CSS 4, Radix UI, Zustand, React Router 7. Located in `ui/web/`. **Use `pnpm` (not npm).**
**Desktop UI:** React 19, Vite 6, TypeScript, Tailwind CSS 4, Zustand, Framer Motion. Located in `ui/desktop/frontend/`. **Use `pnpm`.**
**Desktop App:** Wails v2 (`//go:build sqliteonly`). Located in `ui/desktop/`. Embeds gateway + React frontend in single binary.
**Database:** PostgreSQL 18 with pgvector (standard). SQLite via `modernc.org/sqlite` (desktop/lite). Raw SQL with `$1, $2` (PG) or `?` (SQLite) positional params. Nullable columns: `*string`, `*time.Time`, etc.

## Project Structure

```
cmd/                          CLI commands, gateway startup, onboard wizard, migrations
internal/
├── agent/                    Agent loop (think→act→observe), router, resolver, input guard
├── bootstrap/                System prompt files (SOUL.md, IDENTITY.md) + seeding + per-user seed
├── bus/                      Event bus system
├── cache/                    Caching layer
├── channels/                 Channel manager: Telegram, Feishu/Lark, Zalo, Discord, WhatsApp
│   └── whatsapp/             Native WhatsApp via whatsmeow (v3)
├── config/                   Config loading (JSON5) + env var overlay
├── consolidation/            Memory consolidation workers (episodic, semantic, dreaming) (v3)
├── crypto/                   AES-256-GCM encryption for API keys
├── cron/                     Cron scheduling (at/every/cron expr)
├── edition/                  Edition system (Lite, Standard) with feature gating
├── eventbus/                 Domain event bus with worker pool, dedup, retry (v3)
├── gateway/                  WS + HTTP server, client, method router
│   └── methods/              RPC handlers (chat, agents, sessions, config, skills, cron, pairing)
├── hooks/                    Hook system for extensibility
├── http/                     HTTP API (/v1/chat/completions, /v1/agents, /v1/skills, etc.)
├── i18n/                     Message catalog: T(locale, key, args...) + per-locale catalogs (en/vi/zh)
├── knowledgegraph/           Knowledge graph storage and traversal
├── mcp/                      Model Context Protocol bridge/server
├── media/                    Media handling utilities
├── memory/                   Memory system (pgvector)
├── oauth/                    OAuth authentication
├── orchestration/            Orchestration primitives: BatchQueue[T] generic, ChildResult, media conversion (v3)
├── permissions/              RBAC (admin/operator/viewer)
├── pipeline/                 8-stage agent pipeline (context→history→prompt→think→act→observe→memory→summarize)
├── providers/                LLM providers: Anthropic (native HTTP+SSE), OpenAI-compat (HTTP+SSE), DashScope (Alibaba Qwen), Claude CLI (stdio+MCP bridge), ACP (Anthropic Console Proxy), Codex (OpenAI)
├── providerresolve/          Provider adapter + model registry with forward-compat resolver
├── sandbox/                  Docker-based code execution sandbox
├── scheduler/                Lane-based concurrency (main/subagent/cron)
├── sessions/                 Session management
├── skills/                   SKILL.md loader + BM25 search
├── store/                    Store interfaces + implementations (PostgreSQL, SQLite)
│   ├── base/                 Shared store abstractions: Dialect interface, helpers (NilStr, BuildMapUpdate, BuildScopeClause)
│   ├── pg/                   PostgreSQL implementations (database/sql + pgx/v5)
│   └── sqlitestore/          SQLite implementations (modernc.org/sqlite)
├── tasks/                    Task management
├── tokencount/               tiktoken BPE token counting
├── tools/                    Tool registry, filesystem, exec, web, memory, subagent, MCP bridge, delegate
├── tracing/                  LLM call tracing + optional OTel export (build-tag gated)
├── tts/                      Text-to-Speech (OpenAI, ElevenLabs, Edge, MiniMax)
├── updater/                  Desktop auto-update checker (Lite edition)
├── upgrade/                  Database schema version tracking
├── vault/                    Knowledge Vault with wikilinks, hybrid search, FS sync
├── workspace/                WorkspaceContext resolver for 6 scenarios
pkg/protocol/                 Wire types (frames, methods, errors, events)
pkg/browser/                  Browser automation (Rod + CDP)
migrations/                   PostgreSQL migration files
ui/web/                       React SPA (pnpm, Vite, Tailwind, Radix UI)
ui/desktop/                   Wails v2 desktop app (React frontend + embedded gateway)
```

## Key Patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nextlevelbuilder/goclaw](https://github.com/nextlevelbuilder/goclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
