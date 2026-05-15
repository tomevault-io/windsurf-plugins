---
trigger: always_on
description: > Technical instructions for building n8n-desk. Read this before writing any code.
---

# CLAUDE.md — n8n-desk Build Guide

> Technical instructions for building n8n-desk. Read this before writing any code.

---

## Project State

This is a **greenfield project** — no source code exists yet. The `n8n-master/` directory is a read-only reference copy of the n8n monorepo (gitignored). All spec docs (`PROJECT.md`, `AUTHFLOW_AND_MCPTOOLS.md`, `CHATHUB.md`, `COMPONENT_AND_DESIGN.md`, `WORKFLOW_EMBED.md`) describe what to build. This file describes **how**.

---

## Tech Stack

| Layer | Technology | Notes |
|---|---|---|
| Framework | **Ionic 8 + Vue 3** (`<script setup>`, Composition API) | Single codebase for all platforms |
| Language | **TypeScript** (strict mode) | No `any` — use proper types |
| Build | **Vite** | Via `@ionic/vue` toolchain |
| Desktop | **Electron** (via Capacitor or Electron Forge) | Working directory model for Cowork/Workflow modes |
| Mobile | **Capacitor** (iOS/Android) | Native shell for Ionic app |
| State | **Pinia** | In-memory reactive state, hydrated from local storage |
| Local storage | **`~/.n8n-desk/`** | All persistent data — config, sessions, tokens |
| Routing | **Vue Router** with Ionic's `IonRouterOutlet` | Tab-based top-level nav (Chat/Cowork/Workflow) |
| Agent | **Deep Agents SDK** (`deepagents` + `langchain` + `@langchain/core`) | Local agent for Cowork and Workflow modes |
| HTTP | **ofetch** or **ky** | Lightweight, TypeScript-first fetch wrapper |
| WebSocket | Native WebSocket or **reconnecting-websocket** | For Chat-Hub streaming |
| Styling | **SCSS** with n8n design tokens + Ionic CSS variables | See theming section |
| Testing | **Vitest** (unit), **Cypress** (e2e) | |

---

## Project Structure

```
n8n-desk/
├── CLAUDE.md
├── PROJECT.md, AUTHFLOW_AND_MCPTOOLS.md, etc.
├── package.json
├── tsconfig.json
├── vite.config.ts
├── ionic.config.json
├── capacitor.config.ts
├── index.html
├── src/
│   ├── main.ts                        # App entry, Ionic + Pinia + Router setup
│   ├── App.vue                        # Root component with IonApp
│   ├── router/
│   │   └── index.ts                   # Tab-based routing (Chat/Cowork/Workflow)
│   ├── views/
│   │   ├── OnboardingView.vue         # First-run wizard (connect → login → done)
│   │   ├── ChatView.vue               # Chat mode — thin client to Chat-Hub
│   │   ├── CoworkView.vue             # Cowork mode — local agent + workflow tools
│   │   ├── WorkflowView.vue           # Workflow mode — MCP CRUD agent
│   │   └── SettingsView.vue           # Connection, auth, LLM provider config
│   ├── components/
│   │   ├── chat/                      # Chat UI (forked from n8n AskAssistant)
│   │   │   ├── ChatMessageList.vue
│   │   │   ├── ChatMessage.vue
│   │   │   ├── ChatInput.vue
│   │   │   └── AgentSidebar.vue
│   │   ├── workflow/                  # Workflow visualization
│   │   │   └── WorkflowPreview.vue    # Wraps <n8n-demo> web component
│   │   ├── instance/                  # Instance management
│   │   │   └── InstanceSwitcher.vue   # Header instance indicator + switcher popover
│   │   └── ui/                        # Shared UI components
│   ├── composables/
│   │   ├── useAuth.ts                 # OAuth2 PKCE flow
│   │   ├── useChatHub.ts              # Chat-Hub REST + WebSocket
│   │   ├── useMcp.ts                  # MCP tool calls
│   │   ├── useCoworkAgent.ts          # Cowork mode deep agent (read-only workflows as tools)
│   │   ├── useWorkflowAgent.ts        # Workflow mode deep agent (MCP CRUD tools)
│   │   ├── useConnection.ts           # Connection state (online/offline/reconnecting)
│   │   └── useTheme.ts               # Light/dark theme sync
│   ├── stores/
│   │   ├── instances.ts               # Registered n8n instances, active instance
│   │   ├── auth.ts                    # Token state, user role, scopes (per active instance)
│   │   ├── chat.ts                    # Sessions, messages, agents (hydrated from JSONL)
│   │   ├── workflows.ts              # Workflow cache, search results
│   │   └── settings.ts               # Global app config (hydrated from config.json)
│   ├── services/
│   │   ├── local-storage.ts           # Read/write ~/.n8n-desk/ files (JSONL, JSON)
│   │   ├── n8n-api.ts                 # Base HTTP client (auth headers, refresh)
│   │   ├── chathub.ts                 # Chat-Hub REST endpoints
│   │   ├── chathub-stream.ts          # WebSocket streaming client
│   │   └── mcp.ts                     # MCP tool invocations
│   ├── types/
│   │   ├── chathub.ts                 # Chat-Hub API types (mirrored from n8n)
│   │   ├── mcp.ts                     # MCP tool request/response types
│   │   └── agent.ts                   # Agent message, plan, tool-call types
│   ├── theme/
│   │   ├── n8n-tokens.scss            # Copied from n8n design system
│   │   ├── variables.scss             # Ionic CSS variable overrides
│   │   └── global.scss                # App-wide styles
│   └── utils/
│       ├── markdown.ts                # Markdown rendering (from n8n)
│       └── sanitize.ts                # HTML sanitization (from n8n v-n8n-html)
├── electron/                          # Electron main process (desktop)
│   ├── main.ts                        # App lifecycle, window management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [geckse/n8n-desk](https://github.com/geckse/n8n-desk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
