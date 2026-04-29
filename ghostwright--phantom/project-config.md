---
trigger: always_on
description: Phantom is an autonomous AI co-worker that runs as a persistent Bun process on a VM. It wraps the Claude Agent SDK as a subprocess (Anthropic by default, swappable via a `provider:` config block to Z.AI/GLM-5.1, OpenRouter, Ollama, vLLM, LiteLLM, or any Anthropic Messages API compatible endpoint). It maintains vector-backed memory across sessions, rewrites its own configuration through a validated self-evolution engine, communicates via Slack/Web Chat/Telegram/Email/Webhook, and exposes all capa
---

# Phantom

Phantom is an autonomous AI co-worker that runs as a persistent Bun process on a VM. It wraps the Claude Agent SDK as a subprocess (Anthropic by default, swappable via a `provider:` config block to Z.AI/GLM-5.1, OpenRouter, Ollama, vLLM, LiteLLM, or any Anthropic Messages API compatible endpoint). It maintains vector-backed memory across sessions, rewrites its own configuration through a validated self-evolution engine, communicates via Slack/Web Chat/Telegram/Email/Webhook, and exposes all capabilities as an MCP server. 30,000+ lines of TypeScript, 1,819 tests, v0.20.2. Apache 2.0, repo at ghostwright/phantom.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Runtime | Bun (TypeScript-native, built-in SQLite, no bundler) |
| Agent | Claude Agent SDK (`@anthropic-ai/claude-agent-sdk`) subprocess. Provider is configurable via `src/config/providers.ts`: Anthropic (default), Z.AI, OpenRouter, Ollama, vLLM, LiteLLM, custom. |
| Memory | Qdrant (vector DB, Docker) + Ollama (nomic-embed-text, local embeddings) |
| State | SQLite via Bun (sessions, tasks, metrics, evolution versions, scheduled jobs) |
| Channels | Slack (Socket Mode), Web Chat (SSE streaming), Telegram (long polling), Email (IMAP/SMTP), Webhook (HMAC-SHA256), CLI |
| Chat Client | React 19 + Vite + shadcn/ui + Tailwind v4 SPA at `/chat` |
| Web UI | Tailwind v4 Browser CDN + DaisyUI v5, static files from public/ |
| MCP | Streamable HTTP on /mcp, bearer token auth, 17+ tools |
| Infrastructure | Docker (compose), Specter VMs (Hetzner), systemd (bare metal) |
| Lint/Format | Biome |
| Test | bun:test |

## The Cardinal Rule

**TypeScript is plumbing. The Agent SDK is the brain.**

The Agent SDK (Opus 4.7) has full computer access: Read, Write, Edit, Bash, Glob, Grep, WebSearch, Agent tools. It can understand natural language, read code, explore repos, detect tech stacks, clone repos, install packages, write configs, and reason about anything.

TypeScript handles: starting processes, routing messages, managing sessions, storing data, serving HTTP endpoints, tracking state. Mechanical, deterministic work.

If you find yourself writing a function that does something the agent can do better - STOP. Write a prompt instead.

**Anti-patterns (never do these):**
- `detectJsFrameworks()` - the agent reads package.json and knows
- `parseRepoUrls()` with regex - the agent understands natural language
- `classifyUserIntent()` - the agent understands context
- `extractFactsFromText()` as regex - use LLM judges (already built)
- Structured question state machines - the agent has natural conversations
- Hardcoded detection of languages/databases/tools - the agent reads code

**The only exception:** heuristic fallbacks for when the LLM is unavailable (API down). These are clearly marked with "HEURISTIC FALLBACK" comments in the codebase.

## Build and Test Commands

```bash
bun install                          # Install dependencies
bun test                             # Run 1,819 tests
bun run src/index.ts                 # Start the server
bun run src/cli/main.ts init --yes   # Initialize config (reads env vars)
bun run src/cli/main.ts doctor       # Check all subsystems
bun run src/cli/main.ts status       # Quick one-liner status
bun run lint                         # Biome check
bun run typecheck                    # tsc --noEmit

# Chat UI (separate build)
cd chat-ui && bun install            # Install chat-ui dependencies
cd chat-ui && bun run build          # Build production SPA to chat-ui/dist/
cd chat-ui && bun run typecheck      # Type-check the chat client
cd chat-ui && bun run dev            # Dev server on :5173 (proxy to :3100)

# Chat-UI dev loop: run Phantom on :3100 in one terminal, Vite on :5173 in another.
# Vite proxies /chat/* API calls to :3100. Open http://localhost:5173/chat.
```

## Project Structure

```
src/
  index.ts              # Main entry point. Wires everything together (~500 lines).
  agent/
    runtime.ts          # Claude Agent SDK query() wrapper, session management
    prompt-assembler.ts # System prompt: base + role + evolved + memory context
    hooks.ts            # Safety hooks (dangerous command blocker, file tracker)
    in-process-tools.ts # In-process MCP tool servers (dynamic, scheduler, web UI)
  chat/
    http.ts             # SSE endpoint, session management, message routing
    http-handlers.ts    # Request handlers for chat API routes
    types.ts            # 32-event SSE wire format (discriminated union)
    sdk-to-wire.ts      # Translates Agent SDK events to chat wire frames
    session-store.ts    # In-memory chat session state
    message-store.ts    # Persistent message history (SQLite)
    stream-bus.ts       # Fan-out SSE event bus (multi-tab support)
    upload.ts           # File attachment upload handler

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ghostwright/phantom](https://github.com/ghostwright/phantom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
