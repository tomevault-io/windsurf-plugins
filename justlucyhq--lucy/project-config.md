---
trigger: always_on
description: ﻿# CLAUDE.md -- Lucy AI Platform
---

﻿# CLAUDE.md -- Lucy AI Platform

## Project Overview

Lucy is a multi-provider AI chat platform with a visual workflow builder, dual storage system, and project integration framework. It is built for internal teams and company onboarding -- allowing users to chat with OpenAI, Anthropic, Google, or local (Ollama / LM Studio) models through a unified interface, build AI pipelines with drag-and-drop workflows, and connect Lucy to external apps (like Contractors Room) so the AI has live business context.

Key capabilities:
- **Multi-provider chat** -- stream responses from GPT-4o, Claude, Gemini, or local models (Ollama/LM Studio) through a single UI
- **AI personas** -- 5 built-in system-prompt personas + create custom ones; selected via a chip in the chat input bar
- **Message editing and regeneration** -- inline edit any user message; regenerate any assistant reply
- **Token tracking** -- per-message and per-conversation token estimates shown in the UI
- **Visual workflow builder** -- drag-and-drop node graph for building multi-step AI pipelines
- **Dual storage** -- works standalone with localStorage or connected to Supabase PostgreSQL
- **Integration system** -- external apps register their schema/actions so Lucy can read their data and act on it
- **Auth system** -- Supabase Auth with email/password, Google OAuth, and route-level middleware
- **Embeddable widget** -- one-line script tag to embed Lucy chat in any web app
- **PWA** -- Web App Manifest + three edge-generated icon sizes (32, 192, 512) for installability
- **Docker** -- multi-stage Dockerfile + docker-compose for production container deployment
- **Test suite** -- 219 tests across 33 suites (Jest + React Testing Library)

## Tech Stack

| Technology | Version | Purpose |
|---|---|---|
| Next.js | 16.2.9 | App Router framework (Turbopack), API routes, SSR |
| React | 19.2.7 | UI rendering |
| TypeScript | 5.x | Type safety across the entire codebase |
| Tailwind CSS | 3.4.15 | Utility-first styling, dark/light mode via `class` strategy |
| Zustand | 5.0.2 | Lightweight state management (4 stores: chat, conversations, settings, personas) |
| @xyflow/react | 12.11.0 | React Flow v12 for the workflow canvas |
| Supabase | 2.108.1 | Optional PostgreSQL backend + Auth + RLS |
| OpenAI SDK | 6.42.0 | OpenAI API client (also used for Ollama/LM Studio local endpoints) |
| Anthropic SDK | 0.104.1 | Anthropic API client |
| Google AI SDK | 0.24.1 | Gemini API client |
| react-markdown | 10.1.0 | Markdown rendering in chat messages |
| rehype-highlight | 7.0.0 | Syntax highlighting in code blocks |
| remark-gfm | 4.0.0 | GitHub Flavored Markdown |
| lucide-react | 1.17.0 | Icon library (brand icons removed in 1.x) |
| Jest | 30.4.2 | Test runner (via `next/jest` SWC transform) |
| ESLint | 9.x | Linting via flat config (`eslint.config.mjs`) + eslint-config-next 16 |
| React Testing Library | 16.1.0 | Component testing utilities |
| @testing-library/jest-dom | 6.6.3 | Custom DOM matchers |
| @testing-library/user-event | 14.5.2 | User interaction simulation |

## Architecture

### High-Level Diagram

```
+------------------------------------------------------------------+
|                         Next.js App Router                        |
|                                                                   |
|  app/                                                             |
|  +-- layout.tsx          AuthProvider > StorageProvider > StoreSync > ThemeProvider
|  +-- chat/page.tsx       Main chat UI                             |
|  +-- personas/page.tsx   Persona management                       |
|  +-- workflows/          Workflow list + editor                   |
|  +-- settings/           API keys, local models, theme, data      |
|  +-- auth/               login, signup, forgot-password, callback |
|  +-- api/chat/route.ts   SSE streaming endpoint (rate-limited)    |
|  +-- api/models/route.ts Model list + optional local discovery    |
|  +-- api/embed/route.ts  Embeddable widget script                 |
+------------------------------------------------------------------+
         |                    |                    |
         v                    v                    v
+----------------+  +------------------+  +------------------+
| lib/providers/ |  | lib/storage/     |  | lib/integrations/|
| OpenAI         |  | StorageAdapter   |  | Registry         |
| Anthropic      |  | LocalStorage     |  | Context Builder  |
| Gemini         |  | Supabase         |  | Action Executor  |
| LocalProvider  |  +------------------+  +------------------+
+----------------+           |                    |
         |                   v                    v
         v          +------------------+  +------------------+
+----------------+  | lib/supabase/    |  | lib/workflow/    |
| lib/store/     |  | client.ts        |  | engine.ts        |
| chat.ts        |  | schema.sql       |  | store.ts         |
| conversations  |  | auth.tsx         |  | registry.ts      |
| settings.ts    |  +------------------+  +------------------+
| personas.ts    |
+----------------+

proxy.ts -- route protection (redirects unauthenticated users when Supabase is enabled)
```

### Data Flow: Chat Message

```
User types message
  -> ChatPage.handleSend()

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JustLucyHQ/lucy](https://github.com/JustLucyHQ/lucy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-02 -->
