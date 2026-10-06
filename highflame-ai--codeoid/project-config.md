---
trigger: always_on
description: Identity-first remote control plane for AI coding agents. Powered by ZeroID.
---

# Codeoid

Identity-first remote control plane for AI coding agents. Powered by ZeroID.

## Architecture

```
codeoid start
  ├── Bun.serve() — HTTP + WebSocket server
  ├── ShutdownManager — cleanup registry, signal handlers, 30s grace period
  ├── SessionManager — rate limiting, session resume, scope enforcement
  │   ├── ProviderRegistry — pluggable backends (claude default; openai, gemini, codex, pi, gemini-cli, qwen)
  │   └── Session × N — each drives one provider backend (Claude Agent SDK by default)
  │       ├── ScrollbackBuffer — circular ring, replayed on device handoff
  │       ├── TranscriptStore — JSONL persistence, survives daemon restart
  │       ├── RetryManager — exponential backoff, fallback model
  │       └── AgentIdentityManager — ZeroID identities for agents + sub-agents
  ├── Web UI frontend (SolidJS, served at /ui, also Telegram Mini App)
  └── Telegram frontend (embedded, direct SessionManager access)
```

### Plugin Architecture

Frontends implement the `Frontend` interface and get direct access to the
SessionManager — no network hop for embedded frontends.

```typescript
interface Frontend {
  readonly name: string;
  start(ctx: FrontendContext): Promise<void>;
  stop(): Promise<void>;
}

interface FrontendContext {
  manager: SessionManager;   // direct access
  store: Store;              // audit log
  auth: AuthConfig;          // ZeroID config
  httpServer: Server;        // mount custom routes
  host: string; port: number;
}
```

Register with `daemon.use(new MyFrontend())` — that's it.

### File Structure

```
src/
├── cli.ts                        # CLI entry: start, ls, new, attach, send, etc.
├── config.ts                     # Config: ~/.codeoid/config.json + env vars
├── daemon/
│   ├── server.ts                 # Bun.serve() — HTTP + WebSocket, frontend plugin host
│   ├── session-manager.ts        # Orchestrates N sessions, rate limiting, resume
│   ├── session.ts                # One session: drives a provider backend, retry, scrollback, permissions
│   ├── providers/                # Pluggable agent backends behind one SessionProvider interface
│   │   ├── registry.ts           #   ProviderRegistry + createDefaultProviderRegistry()
│   │   ├── interface.ts          #   SessionProvider contract
│   │   └── claude|openai|gemini|codex|pi|acp|qwen/  # one dir per backend (acp = Gemini CLI)
│   ├── store.ts                  # bun:sqlite — sessions + audit_log tables
│   ├── auth.ts                   # ZeroID JWT verification via @highflame/sdk
│   ├── agent-identity.ts         # ZeroID identities for coding agents + sub-agents
│   ├── scrollback.ts             # Circular ring buffer for device handoff replay
│   ├── transcript.ts             # JSONL persistence for session resume
│   ├── retry.ts                  # Exponential backoff, fallback model, error categories
│   ├── rate-limit.ts             # Per-user session creation + concurrency limits
│   ├── shutdown.ts               # Cleanup registry, signal handlers, grace period
│   └── index.ts
├── protocol/
│   ├── types.ts                  # Client<->Daemon message protocol
│   ├── scopes.ts                 # 8 permission scopes
│   └── index.ts
├── frontends/
│   ├── types.ts                  # Frontend plugin interface
│   ├── index.ts
│   ├── telegram/index.ts         # Grammy bot, embedded in daemon process
│   └── web-ui/index.ts           # Serves the built SolidJS app (web/dist) at /ui, also Telegram Mini App
└── terminal/
    └── client.ts                 # WebSocket client, connects to daemon
```

## Tech Stack

- **Runtime**: Bun (native WebSocket, bun:sqlite, Bun.serve())
- **Agent backends**: pluggable via `ProviderRegistry` — Claude Agent SDK (`@anthropic-ai/claude-agent-sdk`, default), plus OpenAI, Gemini, Codex, pi, the Gemini CLI, and Qwen Code (`@qwen-code/sdk`)
- **Auth**: ZeroID via `@highflame/sdk` (local JWKS verification)
- **Telegram**: Grammy
- **CLI**: Commander
- **Validation**: Zod

No native addon dependencies. Single `bun build` produces a 1.1MB bundle.

## Core Concepts

### Sessions
A session = one agent backend (the Claude Agent SDK by default; also Codex,
Gemini, OpenAI, pi, the Gemini CLI, or Qwen Code) working in one directory.
Sessions are named, persistent, and daemon-owned. Multiple clients can
attach/detach simultaneously from any frontend.

### Auth Model (ZeroID)
Every connection requires a ZeroID JWT. Scopes are enforced per-message:
- `session:create` / `session:destroy` — lifecycle
- `session:list` — discovery
- `session:attach` / `session:watch` — connect to sessions
- `session:send` / `session:interrupt` / `session:approve` — interact

Delegation: users can share scoped tokens (e.g. read-only watcher) with teammates.
Revocation: kill a token -> immediately lose access.
Audit: every action attributed to a ZeroID subject in SQLite.

### Agent Identities
When configured, Claude coding agents and their sub-agents get ZeroID identities:
- Session agent registered on first `send()` (SessionStart hook)
- Sub-agents registered on spawn (SubagentStart hook), with delegated tokens
- Scope attenuation: sub-agents get a subset of parent's scopes
- Cascading revocation: deactivate parent -> all sub-agents revoked

### Device Handoff

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [highflame-ai/codeoid](https://github.com/highflame-ai/codeoid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
