---
trigger: always_on
description: Agent guidance for AI coding assistants (Codex, Copilot, Gemini, etc.) working in this repository.
---

# AGENTS.md

Agent guidance for AI coding assistants (Codex, Copilot, Gemini, etc.) working in this repository.

## Project overview

**HestiaClaw** is a Home Assistant-centric React/Vite AI agent interface. It supports two backend modes selectable from the sidebar:

- **Agent mode** — a native agent harness embedded in the Express server with multi-provider LLM support and config-driven MCP tools
- **N8N mode** — proxies to an N8N AI agent webhook (original backend, still fully supported)

**Stack**: React 19, Vite, Express 5, SQLite (`better-sqlite3` + `connect-sqlite3`), `express-session`, `argon2`, `@anthropic-ai/sdk`, `openai`, `@modelcontextprotocol/sdk`, `neo4j-driver`, plain CSS, `react-markdown`, `remark-gfm`. No TypeScript, no CSS modules, no Tailwind, no automated test suite.

---

## Running the project

```bash
npm install
cp .env.example .env                         # fill in API keys
cp agent.config.example.json agent.config.json   # fill in provider + MCP config
npm run dev                                  # Vite on :5173+ and Express on :3001
npm run build
npm run start
npm run lint
```

### Required env vars

```bash
SESSION_SECRET=replace-with-a-long-random-secret
BOOTSTRAP_ADMIN_USERNAME=admin
BOOTSTRAP_ADMIN_PASSWORD=change-me-now

# Agent mode
OPENAI_API_KEY=...          # or ANTHROPIC_API_KEY for Claude
OPENAI_BASE_URL=            # optional — Ollama, LM Studio, etc.

# N8N mode
N8N_WEBHOOK_URL=https://your-n8n-instance.com/webhook/...

# Optional integrations
ELEVENLABS_API_KEY=...
NEO4J_URI=bolt://...
NEO4J_USER=neo4j
NEO4J_PASSWORD=...
SEARXNG_URL=http://...      # enables built-in web_search tool in agent mode
HA_URL=https://...          # Home Assistant (used by agent.config.json MCP env block)
HA_TOKEN=...
```

---

## Repository structure

```
hestiaclaw/
├── server/
│   ├── index.js                       # Express app — auth, N8N proxy, voice, graph, agent mount
│   └── agent/
│       ├── index.js                   # Agent router (/api/agent/*)
│       ├── loop.js                    # Core agent loop — stream → tool execute → history persist
│       ├── session.js                 # SQLite conversation history (agent_messages table)
│       ├── providers/
│       │   ├── base.js                # Provider interface
│       │   ├── anthropic.js           # Anthropic SDK adapter
│       │   ├── openai.js              # OpenAI SDK adapter (also Ollama/LM Studio)
│       │   └── index.js               # Provider factory
│       ├── tools/
│       │   ├── registry.js            # Tool registry
│       │   └── builtin/
│       │       └── web-search.js      # SearXNG tool (activates if SEARXNG_URL set)
│       └── mcp/
│           └── client.js              # MCP stdio client manager
├── src/
│   ├── App.jsx                        # Root state — sendMessage (N8N) + sendMessageAgent (agent)
│   ├── App.css
│   ├── components/
│   │   ├── ChatMessage.jsx
│   │   ├── Sidebar.jsx                # Conversation list + N8N/Agent mode toggle
│   │   ├── ThinkingAnimation.jsx
│   │   ├── ChatInput.jsx
│   │   ├── LoginScreen.jsx
│   │   ├── AccountPanel.jsx
│   │   └── GraphView.jsx
│   └── lib/voice.js
├── agent.config.json                  # gitignored — copy from agent.config.example.json
├── agent.config.example.json
├── data/                              # gitignored — SQLite files
└── .env.example
```

---

## Agent harness architecture

### Activation

The harness activates when `agent.config.json` exists in the project root. Without it the server logs `"native harness disabled"` and only N8N mode is available.

### Agent config format

```json
{
  "provider": {
    "type": "openai",
    "model": "gpt-4o"
  },
  "systemPrompt": "You are Hestia, a home intelligence assistant...",
  "mcpServers": {
    "home-assistant": {
      "command": "/home/jason/.local/bin/uvx",
      "args": ["hass-mcp"],
      "env": {
        "HA_URL": "${HA_URL}",
        "HA_TOKEN": "${HA_TOKEN}"
      }
    }
  }
}
```

`${VAR}` in `env` values are expanded from `process.env` at startup.

### Provider interface (`server/agent/providers/base.js`)

```js
class Provider {
  get name()                          // 'anthropic' | 'openai'
  async *stream(messages, tools, options)
  // yields: { type: 'token', content }
  //       | { type: 'tool_call', id, name, input }
}
```

Both providers translate their native streaming deltas into this common event shape.

### Agent loop (`server/agent/loop.js`)

1. Load bounded history from SQLite via `session.getContext(conversationId)`
2. Append new user message
3. Call `provider.stream(messages, tools, { system })` — collect `token` and `tool_call` events
4. Emit `token` events to HTTP response as `{"type":"token","content":"..."}` NDJSON lines
5. On `tool_call`: emit `tool_start`, request browser approval for risky write tools, execute via `registry.execute(name, input)`, emit `tool_end`
6. Append assistant turn + tool results to messages; repeat until no tool calls (max 10 iterations)
7. Persist all new messages to SQLite
8. Emit `{"type":"done"}`

Long conversations use bounded context via `session.getContext()`: recent messages are kept verbatim and older messages are summarized into the effective system prompt.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cl0ud6uru/HestiaClaw](https://github.com/cl0ud6uru/HestiaClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
