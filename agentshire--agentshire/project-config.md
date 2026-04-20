---
trigger: always_on
description: > OpenClaw plugin: maps AI Agents as NPCs in a 3D low-poly town.
---

# Agentshire Plugin Architecture Guide

> OpenClaw plugin: maps AI Agents as NPCs in a 3D low-poly town.
> This file is for AI assistants and developers. User docs: [README.md](README.md).

## Tech Stack

- **Runtime**: OpenClaw Plugin SDK (Node.js / ESM)
- **Frontend**: Three.js + vanilla TypeScript (no framework), Vite build, multi-page entries (game / editor / preview / citizen-editor)
- **Protocol**: WebSocket JSON (plugin ↔ frontend), `AgentEvent` / `GameEvent` discriminated unions
- **Audio**: Web Audio API procedural synthesis (zero audio files for ambient) + mp3 BGM
- **Models**: GLTF/GLB, `@gltf-transform` optimization
- **Testing**: Vitest (root + town-frontend each with independent config)

## System Architecture

```
┌──────────────────────────────────────────────────────────────────────────┐
│                            OpenClaw Runtime                              │
│                                                                          │
│   Gateway ─── Agent Sessions ─── Hook System ─── LLM Providers           │
└────────┬──────────────────────────────┬──────────────────────────────────┘
         │ channel.reply                │ 10+ hook callbacks
         ▼                              ▼
┌──────────────────────────────────────────────────────────────────────────┐
│                                                                          │
│  Plugin Layer (Node.js)                               src/plugin/        │
│                                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │   channel    │  │     hook-    │  │   ws-server  │  │    tools     │  │
│  │   inbound    │  │  translator  │  │   WS:55211   │  │  11 AI tools │  │
│  │   dispatch   │  │  Hook→Event  │  │   broadcast  │  │  plan / step │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │     plan-    │  │    editor-   │  │   citizen-   │  │   llm-proxy  │  │
│  │    manager   │  │     serve    │  │  chat-router │  │  2 parallel  │  │
│  │    plan SM   │  │  HTTP:55210  │  │   msg route  │  │  LLM direct  │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                    │
│  │   subagent-  │  │   outbound-  │  │    session-  │                    │
│  │    tracker   │  │    adapter   │  │    history   │                    │
│  │  JSONL watch │  │  media→card  │  │  chat reload │                    │
│  └──────────────┘  └──────────────┘  └──────────────┘                    │
│                                                                          │
└───────┬───────────────────┬──────────────────────────────────────────────┘
        │                   │
        │  WS :55211        │  HTTP :55210
        │  AgentEvent(26+)  │  Editor API / Static Assets
        ▼                   ▼
┌──────────────────────────────────────────────────────────────────────────┐
│                                                                          │
│  Bridge Layer (Browser Bundle)                        src/bridge/        │
│                                                                          │
│  ┌───────────────────────────────────────────────────────────────── ──┐  │
│  │                       DirectorBridge                               │  │
│  │                                                                    │  │
│  │   Phase: idle → summoning → assigning → going_to_office            │  │
│  │          → working → publishing → returning ──→ idle               │  │
│  └────────────────────────────────────────────────────────────────────┘  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │    Event-    │  │    Route-    │  │   Citizen-   │  │   implicit-  │  │
│  │  Translator  │  │   Manager    │  │   Manager    │  │     chat     │  │
│  │   fallback   │  │   A* + ack   │  │  spawn seq   │  │  10 scenes   │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │    State-    │  │   NpcEvent-  │  │   Activity-  │  │   ToolVfx-   │  │
│  │   Tracker    │  │    Queue     │  │    Stream    │  │    Mapper    │  │
│  │  id↔npc map  │  │  bubble prot │  │  500ms flush │  │  tool→anim   │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘  │
│                                                                          │
└───────┬───────────────────────────────────────────────────────┬──────────┘
        │                                                       │
        │  emit GameEvent (65 types)                GameAction  │
        ▼                                           (14 types)  ▲
┌──────────────────────────────────────────────────────────────────────────┐
│                                                                          │
│  Frontend Layer (Three.js + TypeScript)         town-frontend/src/       │
│                                                                          │
│  ┌───────────────────────────────────────────────────────────────────┐   │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Agentshire/Agentshire](https://github.com/Agentshire/Agentshire) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
