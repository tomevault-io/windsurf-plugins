---
trigger: always_on
description: > This file is read by AI coding agents at the start of every session. It is the
---

# WazzapAgents — Developer Context

> This file is read by AI coding agents at the start of every session. It is the
> canonical reference for understanding this project without rediscovery.

---

## Project Overview

**WazzapAgents** is a WhatsApp AI agent system that connects WhatsApp accounts to
an LLM service, enabling automated conversation, moderation, and interactive
features in group and private chats. Post-migration it supports **multiple
accounts** (tenants), one per `folder_path`, each fully isolated (CONTRACT.md §8).

**Tech stack:**
- **Node.js 18+** (ESM/TS) — WhatsApp gateway via Baileys v7; **WebSocket server**
- **Python 3.10+** — LLM bridge with LangChain / ChatOpenAI; **WaSocket client(s)**
- **SQLite** — per-tenant settings, model configs, moderation state, dashboard stats
- **WebSocket** — Node ↔ Python protocol (JSON over WS, CONTRACT.md §1)

> **Reversed topology (post-migration):** the Node gateway is now the WS
> **server** (binds `WS_LISTEN_PORT`, default `3000`); each Python `WaSocket`
> **client** dials it at `NODE_URL` and announces its tenant `folder_path` in a
> `hello`/`hello_ack` handshake. `CONTRACT.md` is the single source of truth for
> the wire protocol (§1), the `make_wa_socket` SDK (§4), and the per-tenant
> folder layout (§8).

**Architecture at a glance:**

```
  phone A        phone B          ← one WhatsApp account per tenant (per-tenant auth)
     ↕              ↕
┌──────────────────────────────────────────────────────────┐
│  Node.js Gateway — WS SERVER, listens on WS_LISTEN_PORT    │
│  src/  (TypeScript)                                        │
│  ├─ server/   WsServer (accept), AccountRegistry (bind)    │
│  ├─ account/  baileysFactory, accountContext,              │
│  │            actionDispatcher, eventForwarder             │
│  │            (one AccountEntry per folder_path: owns its  │
│  │             Database + repositories, no module globals) │
│  ├─ db/       Database + schema/ + repositories/           │
│  ├─ wa/       domain/ + inbound/outbound/actions/          │
│  │            moderation + command/ (CommandRegistry)      │
│  └─ protocol/ types.ts + ports.ts (wire types, CONTRACT §5)│
└──────────────────────────────────────────────────────────┘
        ▲  hello / hello_ack (§1.1)        ▲
        │  incoming_message, whatsapp_status, control        │  actions (Py→Node):
        │  events, acks  (Node→Python)                       │  send_message, react,
   dial │ NODE_URL                                      dial │  delete, kick, …
┌───────┴────────────────┐                  ┌────────────────┴───────┐
│ Python WaSocket A       │                  │ Python WaSocket B       │
│ folder_path=tenants/a   │                  │ folder_path=tenants/b   │
│  wasocket/ (SDK §4)     │                  │  wasocket/ (SDK §4)     │
│  bridge/ AgentSession   │                  │  bridge/ AgentSession   │
│   = composition root,   │                  │   = composition root,   │
│   wiring agent/ collabs:│                  │   wiring agent/ collabs:│
│  ├─ MuteGate            │                  │  ├─ MuteGate            │
│  ├─ BatchProcessor      │                  │  ├─ BatchProcessor      │
│  ├─ Llm1Router          │                  │  ├─ Llm1Router          │
│  ├─ Llm2Responder       │                  │  ├─ Llm2Responder       │
│  ├─ SubAgentCoordinator │                  │  ├─ SubAgentCoordinator │
│  ├─ ReplyDedup          │                  │  ├─ ReplyDedup          │
│  ├─ IdleTrigger         │                  │  ├─ IdleTrigger         │
│  ├─ AckHydrator         │                  │  ├─ AckHydrator         │
│  └─ EventRouter         │                  │  └─ EventRouter         │
└─────────────────────────┘                  └─────────────────────────┘
 <tenants/a>/{auth,db,media,stickers}   <tenants/b>/{auth,db,media,stickers}
            (CONTRACT §8 — fully isolated per tenant)
```

The bridge loads an accounts list (`bridge/accounts.py`) and runs one
`WaSocket`/`AgentSession` per `folder_path`; a single account is the degenerate
case. `AgentSession` is a thin composition root that builds and wires the
injectable `bridge/agent/` collaborators above (no business logic in nested
closures).

---

## Directory Structure

> **Current layout (post-refactor).** The runtime lives in two top-level trees,
> `src/` (Node gateway, TypeScript) and `python/` (bridge + WaSocket SDK), with
> `data/` as the single-account default tenant folder. The earlier staging tree
> no longer exists — its Node and Python subtrees were promoted to `src/` and
> `python/` as the only runtime. `CONTRACT.md` remains the source of truth for
> the wire protocol and per-tenant folder layout.

```
src/                          Node.js gateway runtime (WS SERVER, TypeScript)
  index.ts                    Composition root: config, ws server, per-tenant accounts (wiring only)
  config.ts                   Single config source — all process.env reads (WS_LISTEN_PORT, dirs, DB paths)
  logger.ts                   Structured pino logger
  mediaHandler.ts             Media download from Baileys, validation, path resolution
  server/
    wsServer.ts               WS server: accept clients on WS_LISTEN_PORT, per-connection heartbeat

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Chomosuke9/WazzapAgent](https://github.com/Chomosuke9/WazzapAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
