---
trigger: always_on
description: > **This file is the single source of truth for the LifeOS repository.**
---

# AGENTS.md — LifeOS Telegram Self-Bot

> **This file is the single source of truth for the LifeOS repository.**
> Future AI sessions must read this document first before inspecting source
> files. If the code and this document disagree, the code is authoritative
> and this document must be updated.

---

## 1. Project Overview

LifeOS is a **headless Telegram self-bot** (userbot) that runs as a single
Python `asyncio` process. It uses **Telethon** with a `StringSession`
(never file-based, never interactive) to operate the owner's own Telegram
account. A **FastAPI** micro-server runs in the same process to serve
`/health` (Render/health checks) and a read-only React dashboard.

The user interface is **Glass UI first**: an inline panel system opened via
`Menu` and rendered through an optional helper bot. There is exactly ONE
text command (`Menu`, literal word — no dot prefix); everything else is a
panel/action/input or a natural-language AI request addressed to the
assistant (default trigger `Nova`).

Core subsystems:

1. **RuntimeSupervisor** — the single recovery authority. Owns the self-client
   run loop, heartbeat, keepalive, failsafe, helper bot, profile scheduler,
   web server, and all reconnect/rebuild/full recovery layers.
2. **Save Engine** — Deep Save only (download → re-upload as a NEW Saved
   Messages message) with structured metadata persisted in Supabase (or an
   in-memory fallback).
3. **Profile Engines** — Bio (`about`) and Username (`first_name`), sharing a
   single parameterized `ProfileEngine` and one shared minute-boundary
   scheduler.
4. **AI Runtime** — provider abstraction (OpenAI, Gemini, OpenRouter, Groq,
   Cerebras, Mistral, Dummy), trigger/reply activation, and a tool layer that
   wraps the service layer.
5. **Utility panels** — Retrieve, Delete, Discover (list/find), Database,
   Settings, Health, and Context panels, all reached through `.menu`.

**Tech stack:** Python 3.11 · Telethon · FastAPI · Uvicorn · Supabase
(optional) · React 18 + Vite 5 + Tailwind CSS 3 (dashboard).

---

## 2. Repository Layout (backend)

```
backend/
├── main.py                          # asyncio entry point + crash diagnostics
├── config.py                        # env loader (hard-fail required, default optional)
├── requirements.txt
│
├── runtime/                         # recovery + supervision (see §4)
│   ├── supervisor.py                # RuntimeSupervisor — single recovery authority
│   ├── heartbeat.py                 # 30s snapshot + invariant checks
│   ├── keepalive.py                 # RPC keepalive pings
│   ├── failsafe.py                  # last-resort all-signals-frozen hard reset
│   ├── task_guard.py                # immortal_create_task / guarded_create_task
│   ├── operation_watchdog.py        # guarded_await — operation-level timeout utility
│   ├── tg_retry.py                  # tg_rpc helper (dormant in prod, tested)
│   ├── startup_check.py             # run_startup_checks (dormant in prod, tested)
│   ├── crash_diagnostics.py         # exit-reason + crash snapshot capture
│   ├── diagnostics.py, health_check.py, memory_cleanup.py, states.py, tracer.py
│
├── bot/
│   ├── client.py                    # Telethon self-client factory (StringSession)
│   ├── router.py                    # register_all() wires every handler
│   └── handlers/
│       ├── guard.py                 # is_owner() — single permission gate
│       ├── misc.py                  # Menu (mother panel) + settings/health/context panels
│       ├── save.py                  # Deep Save panels/actions/inputs (no forward)
│       ├── retrieve.py              # saved-items browser panels (under Save)
│       ├── delete.py                # delete panels/actions/inputs
│       ├── discover.py              # list + find panels
│       ├── database.py              # database maintenance panel
│       ├── bio.py                   # Bio profile panels
│       ├── username.py              # Username profile panels
│       ├── ai.py                    # AI config/status panels + AI trigger config inputs
│       └── ai_unified.py            # canonical trigger/reply AI activation
│
├── profile/                         # shared Bio/Username engine
│   ├── engine.py                    # ProfileEngine (parameterized about/first_name)
│   └── scheduler.py                 # ONE shared minute-boundary scheduler
├── bio/engine.py                    # thin Bio wrapper over ProfileEngine
├── username/engine.py               # thin Username wrapper over ProfileEngine
│
├── services/                        # business logic (handlers/tools delegate here)
│   ├── save_service.py              # execute_save (Deep Save pipeline) + link save
│   ├── retrieve_service.py, delete_service.py, discover_service.py,
│   ├── database_service.py, settings_service.py, organize_service.py
│   ├── bio_service.py, username_service.py
│
├── ai/                              # AI runtime (providers, tools, conversation, memory)
│   ├── providers/                   # factory + registry + per-provider modules
│   ├── engine/                      # engine, dispatcher, hooks, metrics
│   ├── tools/                       # Tool base, registry, executor, per-domain tools

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Onlyicing1/Telegram-self-bot](https://github.com/Onlyicing1/Telegram-self-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
