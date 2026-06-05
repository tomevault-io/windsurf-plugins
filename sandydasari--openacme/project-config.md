---
trigger: always_on
description: TypeScript agent platform. Multi-provider LLM with streaming tool-calls, SQLite-backed sessions, MCP integration, OAuth (ChatGPT / Claude subscriptions), an Ink-based CLI TUI, and a Next.js web UI served by the Hono server.
---

# OpenAcme — Claude Code guide

TypeScript agent platform. Multi-provider LLM with streaming tool-calls, SQLite-backed sessions, MCP integration, OAuth (ChatGPT / Claude subscriptions), an Ink-based CLI TUI, and a Next.js web UI served by the Hono server.

**Design lens.** This is an *AI workforce* platform — a structured set of role-specialized agents working for a small human team — not a single-agent assistant. Each agent carries a `name`, a `role` (third-person paragraph for coworkers), and a `persona` (second-person system prompt), and owns its own config, model, tools, skills, MCP servers, sessions, tasks, memory, workspace, and resources. When designing new state or behavior, the question is "does this hold for N agents working in parallel under different roles?" — not "does this work for the agent." Anywhere you'd reach for a global table or a singleton, default to scoping by `agent_id` instead. Cross-agent work — one agent assigning a task to another, one agent looking up coworkers via `agent_list` — is a first-class primitive, so anything you add should be addressable by agent.

---

## Workspace layout

Turborepo + pnpm 9. Workspace globs: `apps/*`, `packages/*`. Published packages are `@openacme/*`; internal tooling is `@repo/*`.

```
apps/
  cli/          # `openacme` binary — Commander + Ink TUI + Clack setup
  web/          # Next.js 16 chat/agents/skills UI; static-built into packages/server/web
  docs/         # Next.js docs site (placeholder)

packages/
  agent-core/   # Agent class — agentic loop, streaming, history reconstruction
  server/       # Hono HTTP server + AgentManager (multi-agent orchestration)
  cli (apps)    # see above
  llm-provider/ # getModel() — OpenAI / Anthropic / Google / OpenRouter / Ollama / custom
  mcp-client/   # MCP stdio + HTTP/SSE transports; tool discovery into registry
  tools/        # ToolRegistry + built-ins (shell, read_file, write_file, edit,
                #   apply_patch, list_files, search_files, session_search, skill_view,
                #   web_search, web_extract, execute_code, process, memory, task_*,
                #   agent_list, browser_*)
  browser/      # Per-agent browser sessions via pluggable provider (local Chrome / Browserbase / Browser-Use / Firecrawl)
  db/           # better-sqlite3 + Drizzle; sessions/messages/user_profiles +
                #   task_comments/task_events + FTS5 (agents are filesystem-backed
                #   under <dataDir>/agents/, not in the DB)
  memory/       # Per-agent persistent MEMORY.md store (Anthropic memory_20250818 +
                #   Claude Code index/topic-file convention)
  tasks/        # Workforce task store (filesystem; isolation by assignee)
  config/       # Zod schema + YAML/JSON loader (~/.openacme/config.yaml)
  auth/         # OAuth (ChatGPT subscription, Claude Pro), token store, body/response
                #   transforms, refresh
  skills/       # SKILL.md discovery, progressive disclosure + multi-source hub
                #   (GitHub, marketplaces, URL, well-known, local, builtin) under hub/;
                #   bundled skills live at packages/skills/builtin/<name>/
  agent-catalog/# Bundled agent templates (Software Engineer, …) — Importable via web or CLI
  ui/           # Shared React components (minimal)
  eslint-config, typescript-config   # @repo/* internal
```

Default data dir: `~/.openacme/` (`config.yaml`, `auth.json` mode 0600, `state.db`, `AGENTS.md`, `agents/<id>/{AGENT.md,workspace/,resources/,memory/,browser-profiles/{chromium,firefox}/}`, `tasks/<id>.md`, `skills/`).
Default server: `127.0.0.1:3210`. Default model: `openrouter` + `anthropic/claude-sonnet-4-20250514`.

---

## The agent loop — request path

User message → response, end-to-end. **SSE is the only delivery channel for an agent turn** — interactive and autonomous turns share one streaming model. The originating tab is just another subscriber to the per-session SSE channel.

1. **Web** `apps/web/app/page.tsx` manages `messages` state directly; `useLiveSession` opens an `EventSource` to `/api/sessions/:id/stream` keyed on `activeSessionId` and waits for `connected: true` before posting. For a fresh chat the client mints sessionId + user-message id (`crypto.randomUUID`) so the SSE subscription can open BEFORE the first POST.
2. **POST** `/api/chat` with `{ agentId, sessionId, messages: UIMessage[] }`:
   - Validates pending FileUIPart URLs, then `commit()`s them (moves bytes from `<dataDir>/attachments/__pending__/` to `<sessionId>/<attId>/<filename>`) and rewrites each part's URL.
   - Provider-gates non-text parts via `lookupModelMetadata(...).inputModalities`.
   - Ensures the session row exists with the caller-supplied id; persists + broadcasts the user message (`messages_appended`).
   - Stores an `AbortController` in the per-session `activeTurns` map.
   - Kicks off `runChatTurn(...)` in the **background** and returns `{ sessionId, userMessageId, assistantMessageId }` JSON immediately.
3. **`runChatTurn`** (helper at bottom of `app.ts`):
   - Marks interactive-busy, broadcasts `session_state: running`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sandydasari/openacme](https://github.com/sandydasari/openacme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
