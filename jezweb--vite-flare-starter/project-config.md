---
trigger: always_on
description: The starter ships **four kinds of agent**, all built on Cloudflare's
---

# Agent architecture

The starter ships **four kinds of agent**, all built on Cloudflare's
`agents` SDK. Pick the right base for what you're building — they're
not interchangeable.

> **Building "an agent that watches X periodically and surfaces findings"?**
> Don't subclass `AutonomousAgent` for it. Use a **Routine** —
> declarative config (agent + schedule + tools allow-list + skills +
> hooks) on top of an existing `AutonomousAgent`. See
> [`ROUTINES.md`](./ROUTINES.md) for the canonical pattern. Issue #50
> decision F: Routines is the user-facing pattern; `scheduled-agents`
> and `webhook-agents` stay as the lower-level primitives.

```
Agent (from agents SDK)              ← all stateful long-lived things
│
├── LiveAgent (via withVoiceInput)   ← live WebSocket session (Voice / Video)
│
├── ReminderAgent                    ← scheduled task using SDK schedule()
│   (extends Agent directly)
│
├── AIChatAgent (SDK class)          ← multi-session chat surface
│   └── ChatAgent                    ← worked: the chat module (shipped, #34)
│
├── AutonomousAgent                  ← stateful AI with persona + memory + tools
│   (in this starter)
│   ├── AssistantAgent               ← worked: per-user persistent assistant
│   ├── ResearcherAgent              ← worked: web_search + delegate_to_writer
│   └── WriterAgent                  ← worked: prose composer (handoff target)
│
└── McpAgent (SDK class)             ← agent exposed AS an MCP server
    └── ScratchpadMcpAgent           ← worked: per-user scratchpad over MCP
```

## Decision matrix

| If you need... | Use... | Worked example |
|---|---|---|
| Live mic / camera / WebSocket session per user | `Agent` + `withVoiceInput` (or `withVideoInput`) mixin | `VoiceInputExample`, `VideoInputExample` |
| Scheduled fire (one-shot or recurring) for non-AI work | `Agent` directly + `this.schedule()` / `this.scheduleEvery()` | `ReminderAgent` |
| Stateful AI assistant with persona + memory + tools | `AutonomousAgent` | `AssistantAgent` |
| Multi-agent handoff (specialist agents call each other) | `AutonomousAgent` + `delegate_to_X` tool on `this.runAgentTool` (awaited or detached facet child) | `ResearcherAgent` → `WriterAgent` |
| Expose agent's data over MCP for external clients | `McpAgent` from `agents/mcp` (SDK) + `McpServer` from `@modelcontextprotocol/sdk` | `ScratchpadMcpAgent` |
| Multi-session AI chat with state-sync to clients | `AIChatAgent` from `agents/chat` (SDK) | `ChatAgent` in `src/server/modules/chat/chat-agent.ts` (shipped — closed issue #34) |
| Long-running multi-step business logic with checkpointing | Cloudflare Workflows + `AgentWorkflow` from `agents/workflows` | _not yet shipped_ |
| High-throughput async fan-out | Cloudflare Queues | _not yet shipped_ |
| Single account-wide cron | `wrangler.jsonc` `triggers.crons` | the `*/15 * * * *` healthcheck |
| **Task-running agent where Anthropic owns the loop** | [`cloudflare/claude-managed-agents`](https://github.com/cloudflare/claude-managed-agents) template | — (separate repo, not this starter) |

**Don't reach for raw `DurableObject`.** Every long-lived stateful thing
in this starter extends `Agent` from the SDK so we get state sync,
schedule/queue/retry, hibernation, RPC, MCP client, and observability
without re-implementing them. The one time we hand-rolled this
(commit 759207a, deleted in f8d646f) we re-invented every wheel and
shipped −332 net lines of code by deleting the work.

### vite-flare-starter vs Claude Managed Agents

[Cloudflare announced Claude Managed Agents](https://blog.cloudflare.com/claude-managed-agents/)
in May 2026 — a deployment pattern where **Anthropic hosts the agent
loop** (model + reasoning + tool-call orchestration) and **Cloudflare
hosts the sandbox + tools**. It's not a competitor to this starter;
it's an *alternative deployment shape* for a different product
shape. The two are complementary.

| | vite-flare-starter | Claude Managed Agents |
|---|---|---|
| Agent loop | Self-hosted (`AutonomousAgent` + AI SDK v6) | Anthropic-managed |
| Tools | `ToolDefinition` in `src/server/modules/chat/tools/` | `defineTool({ name, inputSchema, run })` in `custom-tools.js` |
| Sandbox | `@cloudflare/sandbox` already bound | Same primitive |
| Persistence | DO storage + D1 projection + R2 | Anthropic-managed state |
| Multi-tenancy | Per-(user, conv) DO instance | Anthropic-managed |
| MCP | Native — agent inherits user's MCP tools | Via custom tools |
| Customisation ceiling | Full (we own the loop) | Constrained to template + custom tools |
| **Pick when** | Building a SaaS product (chat UX, projects, orgs, voice, skills, memories) | Building a task-running agent fast ("hey Claude, do X") |

Their tool shape `defineTool({ name, inputSchema: z.object(...), run })`
is nearly identical to our `ToolDefinition` contract — independent
convergence on the same primitive is good validation. If you ever need
to expose this starter's tools to a managed agent, the adapter is
~20 lines (map `ToolDefinition.execute` → `defineTool.run`).

## AutonomousAgent — the AI agent base

`src/server/lib/agents/autonomous-agent.ts`

A subclass-and-go base for "AI entity with identity, memory, tools, and
autonomous triggers." Everything below this line is what subclasses get
for free.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jezweb/vite-flare-starter](https://github.com/jezweb/vite-flare-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
