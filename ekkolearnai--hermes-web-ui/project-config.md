---
trigger: always_on
description: Hermes Web UI is a web dashboard for [Hermes Agent](https://github.com/EKKOLearnAI/hermes-web-ui), a multi-platform AI chat system. It provides session management, scheduled jobs, usage analytics, model configuration, channel management (Telegram, Discord, Slack, WhatsApp, etc.), an integrated terminal, and a streaming chat interface.
---

# CLAUDE.md — Hermes Web UI Development Guide

## Project Overview

Hermes Web UI is a web dashboard for [Hermes Agent](https://github.com/EKKOLearnAI/hermes-web-ui), a multi-platform AI chat system. It provides session management, scheduled jobs, usage analytics, model configuration, channel management (Telegram, Discord, Slack, WhatsApp, etc.), an integrated terminal, and a streaming chat interface.

The project is designed for **multi-agent extensibility** — Hermes is the first agent integration. All agent-specific code is namespaced under `hermes/` directories, so future agents can be added alongside without conflicts.

**Tech stack:**

- **Frontend:** Vue 3 (Composition API, `<script setup lang="ts">`), Naive UI, Pinia, vue-router (hash history), vue-i18n, SCSS, Vite
- **Backend:** Koa 2, @koa/router v15+, node-pty (WebSocket terminal), reverse proxy to Hermes gateway
- **Language:** TypeScript (strict mode), single package (no workspaces)

---

## Development Commands

```bash
npm run dev           # Start both server (nodemon) and client (Vite) concurrently
npm run dev:client    # Vite dev server only (proxies API to backend)
npm run dev:server    # nodemon + ts-node for server only
npm run build         # Type-check (vue-tsc) -> Vite build -> tsc server build
npm run preview       # Preview production build with Vite
```

- **Dev port:** 8648 (client Vite dev server proxies `/api`, `/v1`, `/health`, `/upload`, `/webhook` to `http://127.0.0.1:8648`)
- **Prerequisite:** `hermes` CLI must be installed and on `$PATH` (the server wraps it via `child_process.execFile`)

---

## Project Structure

```
hermes-web-ui/
├── bin/                          # CLI entry point (bin/hermes-web-ui.mjs)
├── dist/                         # Build output
│   ├── client/                   # Vite frontend build
│   └── server/                   # tsc server build
├── packages/
│   ├── client/src/               # Vue 3 frontend
│   │   ├── api/                  # API layer
│   │   │   ├── client.ts         # Shared: base request utility (auth, fetch wrapper)
│   │   │   └── hermes/           # Hermes-specific API modules
│   │   │       ├── chat.ts       # Gateway proxy: runs, SSE events, models
│   │   │       ├── jobs.ts       # Gateway proxy: scheduled jobs CRUD
│   │   │       ├── sessions.ts   # Local BFF: session management (wraps hermes CLI)
│   │   │       ├── config.ts     # Local BFF: app config, weixin credentials
│   │   │       ├── logs.ts       # Local BFF: log file listing & reading
│   │   │       ├── skills.ts     # Local BFF: skills listing, memory CRUD
│   │   │       └── system.ts     # Local BFF: health, model config, providers
│   │   ├── components/           # Vue components
│   │   │   ├── layout/           # Shared: AppSidebar, LanguageSwitch, ModelSelector
│   │   │   └── hermes/           # Hermes-specific components
│   │   │       ├── chat/         # ChatPanel, ChatInput, MessageList, MarkdownRenderer
│   │   │       ├── jobs/         # JobCard, JobFormModal, JobsPanel
│   │   │       ├── models/       # ProviderCard, ProviderFormModal, ProvidersPanel
│   │   │       ├── settings/     # AgentSettings, DisplaySettings, MemorySettings, etc.
│   │   │       ├── skills/       # SkillList, SkillDetail
│   │   │       └── usage/        # StatCards, DailyTrend, ModelBreakdown
│   │   ├── i18n/locales/         # en.ts, zh.ts
│   │   ├── router/index.ts       # vue-router (hash history)
│   │   ├── stores/               # Pinia stores
│   │   │   └── hermes/           # Hermes-specific stores
│   │   │       ├── app.ts        # App-level state (health, sidebar, models)
│   │   │       ├── chat.ts       # Chat sessions, messages, streaming
│   │   │       ├── jobs.ts       # Scheduled jobs CRUD
│   │   │       ├── models.ts     # Model provider management
│   │   │       ├── settings.ts   # App configuration
│   │   │       └── usage.ts      # Usage statistics
│   │   ├── styles/               # global.scss, variables.scss
│   │   └── views/                # Page-level components
│   │       ├── LoginView.vue     # Shared: login page
│   │       └── hermes/           # Hermes-specific pages
│   │           ├── ChatView.vue
│   │           ├── JobsView.vue
│   │           ├── ModelsView.vue
│   │           ├── LogsView.vue
│   │           ├── UsageView.vue
│   │           ├── SkillsView.vue
│   │           ├── MemoryView.vue
│   │           ├── SettingsView.vue
│   │           ├── ChannelsView.vue
│   │           └── TerminalView.vue
│   ├── server/src/               # Koa BFF server
│   │   ├── routes/hermes/        # Route modules
│   │   │   ├── index.ts          # Aggregates all hermes sub-routers
│   │   │   ├── sessions.ts       # Session CRUD (wraps hermes CLI)
│   │   │   ├── profiles.ts       # Profile management (wraps hermes CLI)
│   │   │   ├── config.ts         # App config read/write
│   │   │   ├── filesystem.ts     # Skills, memory, model config, providers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EKKOLearnAI/hermes-web-ui](https://github.com/EKKOLearnAI/hermes-web-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
