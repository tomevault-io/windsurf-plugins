---
trigger: always_on
description: Hi. If you're an AI coding agent (Claude Code, Codex, Cursor, anyone) about to make changes here, read this once before you touch the source.
---

# AGENTS.md — Guardrails for AI coding agents working on iClaw

Hi. If you're an AI coding agent (Claude Code, Codex, Cursor, anyone) about to make changes here, read this once before you touch the source.

## What iClaw is

A **local web UI for [OpenClaw Gateway](https://docs.openclaw.ai)**. Nothing more. The app:

- Renders a ChatGPT-style chat surface (sidebar of chats, conversation in the middle).
- Stores chat history locally in SQLite (`data/iclaw.db`).
- Talks to a running OpenClaw Gateway on the same machine via the **native WebSocket protocol** (no HTTP chat completions, no SSE).
- Auto-loads the bearer token from `~/.openclaw/openclaw.json`.
- Adds higher-level features that the gateway doesn't have natively: per-project shared facts, scheduled messages, slash autocomplete, header model picker, today's cost chip, exec approval cards.

## What iClaw is **not**

These are intentional non-goals. Don't add them. If a request lands here that needs any of them, push back or open an issue first.

- **Not a generic OpenAI-compat client.** We migrated off `/v1/chat/completions` in commit `a92c10e`. Do not re-introduce an HTTP chat completion path or SSE-based streaming — everything chat-related is native WS.
- **Not a multi-provider chat tool.** No fallback to Ollama, vLLM, Claude API, OpenAI direct, etc. OpenClaw is the only backend; switch model providers in `openclaw.json`.
- **Not a remote/hosted product.** Loopback-only. No multi-user, no auth layer, no team features. Threat model is "this user on this machine".
- **For the default Full Power chat, not a re-implementation of OpenClaw's agent runtime.** On that path we don't compact transcripts, run tools, or wrap the LLM — we display what the gateway produces and (separately) maintain a project-scoped fact layer *additional* to OpenClaw memory. (The Work / Safe work / Incognito modes are the deliberate exception — a small, self-contained agent loop in `packages/iclaw-runtime`; see below. Don't grow it into a general OpenClaw replacement.)
- **Not heavyweight on the frontend.** No build step beyond `tsc`. Plain CSS, vanilla JS on the client, EJS for views. If you want to add React/Vue/Svelte, open an issue first — the answer is probably no for the current scope.

## Architecture you need to know

```
                  ┌─────────────────┐
                  │  OpenClaw       │
                  │  Gateway        │
                  │  (port 18789)   │
                  └────┬────────┬───┘
                       │WS      │HTTP /api/chat/media/*
                       │native  │(proxied through /media)
                  ┌────▼────────▼─────────────────────────┐
                  │  iClaw Express + WS app (port 3000)   │
                  │                                       │
                  │  gatewayWs.ts    ← single WS bridge,  │
                  │      ▲           ← tick watchdog,     │
                  │      │           ← onReconnect()      │
                  │  openclawWs.ts   ← typed RPC client   │
                  │      ▲                                │
                  │      ├─ chatRunner.ts (per-turn)      │
                  │      ├─ gatewayEvents.ts (global)     │
                  │      ├─ scheduler.ts (cron sweep)     │
                  │      └─ projectMemory.ts (facts)      │
                  │      ▲                                │
                  │      ├─ routes/ws.ts (browser WS)     │
                  │      ├─ routes/chats.ts (HTTP forms)  │
                  │      ├─ routes/projects.ts            │
                  │      └─ routes/gateway.ts (proxies)   │
                  │                                       │
                  │  wsHub.ts ← pub/sub fanout            │
                  └────┬───────────────────────────────┬──┘
                       │WS /ws (real-time)             │HTTP (forms + pages)
                       ▼                               ▼
                  ┌────────────────────────────────────────┐
                  │  Browser — public/js/iclaw.js +        │
                  │  vendored marked.min.js + highlight.js │
                  └────────────────────────────────────────┘
```

Browser ↔ server is **one persistent WebSocket** at `/ws`. Chat traffic (send, abort, exec approval, streaming turn events, cross-tab sync) flows through it. HTTP routes exist for page rendering (EJS), form actions, the `/media/*` proxy, and JSON endpoints under `/api/gateway/*`. Every HTTP mutation that touches a chat also emits the matching `chat-updated` / `chat-deleted` over WS so other tabs catch up instantly.

### Runtime modes (Work / Safe work / Incognito)

The default **Full Power** mode goes to the gateway (above). The other three modes
are served by the bundled **iClaw runtime** (`packages/iclaw-runtime`), a small
agent runtime kept deliberately minimal:

- The host calls it over HTTP on `127.0.0.1:7430` (`src/services/workRuntime.ts` →
  runtime `index.ts`). The model loop runs **on the host** via OpenRouter.
- Tool/shell execution is isolated in a **Docker sandbox**, one model per file:
  `secure-runner.ts` (Safe work / Incognito — everything runs in a per-turn

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iClawApp/iClaw](https://github.com/iClawApp/iClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
