---
trigger: always_on
description: This file provides guidance for AI agents operating in this repository.
---

# AGENTS.md - Agent Coding Guidelines for AgentHub

This file provides guidance for AI agents operating in this repository.

---

## 0. Documentation Authority

- Agent behavior, feature design, architecture decisions, and implementation details must be constrained by the repository documentation in `docs/`.
- Before each development round, read the relevant design document(s) in `docs/` first. If no matching document exists, create or update the closest documentation entry before or alongside implementation.
- If implementation and documentation disagree, do not silently choose one. Tell the user what is stale or inconsistent, explain the impact briefly, and ask whether the documentation should be revised.
- When a change intentionally updates behavior, contracts, module boundaries, architecture, permissions, orchestration, or artifact protocols, update the matching docs in the same work batch unless the user explicitly says not to.
- Keep documentation practical and current. Prefer concise specs, API contracts, ADRs, and module notes that future agents can use directly.

---

## 1. Repository Structure

AgentHub is a multi-agent collaboration platform built around IM-style interaction.

```
AgentHub/
├── web/              # React + Vite frontend
├── hub-server/       # Node/Bun + Hono platform backend
├── agent-runtime/    # Node/Bun + Hono agent sidecar runtime
├── docs/             # Product, architecture, contracts, and ADRs
└── .agents/          # Local agent skills and tool references
```

### `web/`

- Provides the web client and primary user experience.
- Owns IM-style UI: conversation list, single-agent chats, group chats, message stream, artifacts, previews, and editing entry points.
- Development runs with `cd web && bun dev`.
- Production web assets are expected to be integrated into `hub-server` and exposed through the web port.
- Future migration to an Electron client should preserve web app boundaries and avoid coupling UI directly to local-only capabilities.

### `hub-server/`

- Provides platform APIs using Node/Bun + Hono.
- Owns user-facing backend concerns such as sessions, conversations, messages, agent registry, artifact metadata, and web asset hosting.
- Acts as the single backend entry point from `web`.
- Communicates with `agent-runtime` for AI execution and orchestration.

### `agent-runtime/`

- Runs as a **Sidecar process** for the AgentHub application (Web + HubServer).
- In production, HubServer automatically spawns agent-runtime at startup and manages its lifecycle (health check, auto-restart, graceful shutdown).
- In development, agent-runtime can be started independently for debugging and hot-reload.
- Owns all AI execution concerns: LLM calls, external agent adapters, orchestration, tool calls, permissions, sandbox policy, and artifact generation.
- The frontend must not call LLM providers directly or hold LLM credentials.
- External agent integrations such as Claude Code, Codex, OpenCode, and custom agents must go through this layer.
- See `docs/adr/ADR-001-sidecar-architecture.md` for architectural decision.

---

## 2. Runtime Boundaries

- Required flow: `web -> hub-server -> agent-runtime`.
- `web` should call `hub-server` APIs only.
- `hub-server` should coordinate product state and delegate AI execution to `agent-runtime`.
- `agent-runtime` is a Sidecar process managed by `hub-server` (see `docs/adr/ADR-001-sidecar-architecture.md`).
- `agent-runtime` should expose controlled runtime APIs to `hub-server`, not directly to browser UI.
- LLM credentials, provider adapters, command execution, file access, deployment, and network-sensitive operations belong in `agent-runtime`.
- When adding or changing Agent Runtime API behavior, update `docs/contracts/AGENT_RUNTIME_API_CONTRACTS.md` and the relevant architecture doc.

---

## 3. Build, Test, and Run Commands

Use Bun for JavaScript/TypeScript package execution in this repository.

### Root

```bash
bun run dev:web
bun run dev:server
```

### Web

```bash
cd web && bun dev
cd web && bun run lint
cd web && bunx tsc --noEmit -p tsconfig.app.json
```

### Hub Server

```bash
cd hub-server && bun dev
```

### Agent Runtime

```bash
cd agent-runtime && bun dev
```

`agent-runtime` may not have its runnable scaffold yet. When implementing it, add a local `package.json` and document the commands in `docs/architecture/AGENT_RUNTIME.md`.

Note: In production, agent-runtime is automatically started by hub-server as a Sidecar process. The `bun dev` command is for development/debugging only.

### Verification Policy

- At the end of each coding task, prefer lightweight checks: type checks, lint, focused tests, API smoke tests, or targeted runtime checks.
- Avoid `build` commands by default because they are heavier in time and resources.
- Use build commands only when the user asks, when preparing release verification, or when the changed behavior cannot be validated responsibly without a build.
- If a relevant test command does not exist yet, note that clearly and add a focused one when the task justifies it.

---

## 4. Product Priorities

### P0

- IM-style chat shell.
- Conversation list and multi-session workflow.
- Single-agent conversation.
- Basic message stream and persisted context.

### P1


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GuqierMcl/AgentHub](https://github.com/GuqierMcl/AgentHub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
