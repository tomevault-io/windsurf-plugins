---
trigger: always_on
description: Pinchy is an **enterprise AI agent platform** built on top of [OpenClaw](https://github.com/openclaw/openclaw). OpenClaw is the most powerful open-source AI agent runtime — but it's designed for individual power users. Pinchy adds the enterprise layer: permissions, audit trails, user management, and governance.
---

# CLAUDE.md — Pinchy

## What is Pinchy?

Pinchy is an **enterprise AI agent platform** built on top of [OpenClaw](https://github.com/openclaw/openclaw). OpenClaw is the most powerful open-source AI agent runtime — but it's designed for individual power users. Pinchy adds the enterprise layer: permissions, audit trails, user management, and governance.

**Status: Early development.** The core is working — setup wizard, authentication, provider configuration, agent chat via OpenClaw, agent permissions (allow-list model), knowledge base agents, user management with invite system, personal and shared agents, per-user/org context management, Smithers onboarding interview, audit trail, Telegram channel integration, and Docker Compose deployment. Enterprise features (granular RBAC, plugin marketplace, additional channel integrations) are next.

### The Problem Pinchy Solves

Companies want AI agents but face a trilemma:
- **Cloud platforms** (Dust, Glean, Copilot Studio) → data leaves your servers. Non-starter for EU regulated industries.
- **Workflow builders** (n8n, Dify) → chain steps visually, but not autonomous agents.
- **Frameworks** (CrewAI, LangChain) → libraries, not platforms. No UI, no permissions, no deployment.
- **OpenClaw** → best agent runtime, but no multi-user, no RBAC, no audit trail.

### Target Architecture (PARTIALLY IMPLEMENTED)

```
┌─────────────────────────────────────────┐
│              Pinchy Platform             │
│                                         │
│  ┌──────────┐  ┌──────────┐  ┌───────┐ │
│  │ Web UI   │  │ REST API │  │ Admin │ │
│  └────┬─────┘  └────┬─────┘  └───┬───┘ │
│       │              │            │     │
│       │  ┌───────────────────┐    │     │
│       │  │ Channels          │    │     │
│       │  │ (Telegram, …)     │    │     │
│       │  └────────┬──────────┘    │     │
│       │           │               │     │
│  ┌────┴───────────┴───────────────┴───┐ │
│  │         Permission Layer           │ │
│  │  (RBAC, Scoped Tools, Audit Log)   │ │
│  └────────────────┬───────────────────┘ │
│                   │                     │
│  ┌────────────────┴───────────────────┐ │
│  │        OpenClaw Runtime            │ │
│  │  (Agents, Sessions, Channels,      │ │
│  │   Plugins, MCP, Memory)            │ │
│  └────────────────────────────────────┘ │
│                                         │
│  🔌 Plugin Architecture                │
│  🔐 Role-Based Access Control          │
│  📋 Audit Trail (IMPLEMENTED)          │
│  💬 Telegram Integration (IMPLEMENTED) │
│  🔀 Cross-Channel Workflows            │
│  🏠 Self-Hosted & Offline-Capable      │
│  🤖 Model Agnostic (OpenAI, Anthropic, │
│     Ollama, local models)              │
└─────────────────────────────────────────┘
```

### Core Concepts (planned and implemented)

- **Plugin Architecture** (partially implemented): Agents get scoped tools, not raw shell access. Two plugins implemented: `pinchy-files` (read-only file access for Knowledge Base agents) and `pinchy-context` (saves user/org context during Smithers onboarding interview). Plugin marketplace is planned.
- **Agent Permissions** (implemented): Allow-list model — agents start with zero tools, admins grant specific capabilities. Safe tools (list/read approved dirs) vs. powerful tools (shell, write, web).
- **RBAC** (partially implemented): Admin/user roles with agent access control (admins see all, users see shared + personal agents). Granular per-team/per-role RBAC is planned.
- **Audit Trail** (implemented): Every admin action logged — who, what, when. HMAC-SHA256 signed rows, integrity verification, CSV export. Compliance-ready.
- **User Management** (implemented): Invite system with token-based onboarding, admin and user roles, password management.
- **Knowledge Base Agents** (implemented): Scoped read-only access to specific directories. Template-based creation.
- **Smithers Onboarding** (implemented): New users get an onboarding interview — Smithers learns about them through conversation and saves their context via plugin tools. Admins are additionally asked about their organization.
- **Telegram Channels** (implemented): Admins set up Telegram in Settings → Telegram (guided flow with BotFather instructions, connects to Smithers). Additional agents can be connected via Agent Settings → Channels. Users link their Telegram account by scanning a QR code, messaging the bot, and entering a pairing code. Sessions are unified across web and Telegram via `identityLinks`. Config architecture: DB is source of truth, `regenerateOpenClawConfig()` writes the config file (both at startup and from routes after changes). OpenClaw detects file changes via internal file watcher and hot-reloads. No WebSocket RPC (`config.patch`) needed for config changes.
- **Cross-Channel Workflows**: Additional channels (email, Slack) and cross-channel routing are planned. Telegram is the first implemented channel.
- **Self-Hosted**: Your server, your data, your models. Works without internet.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [heypinchy/pinchy](https://github.com/heypinchy/pinchy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
