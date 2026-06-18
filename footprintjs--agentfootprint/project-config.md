---
trigger: always_on
description: This is the **agentfootprint** library — a framework for building Generative AI applications where context engineering is buildable at the control-flow level. Built on [footprintjs](https://github.com/footprintjs/footPrint) (the flowchart pattern for backend code).
---

# agentfootprint — Agent Instructions (OpenAI Codex)

This is the **agentfootprint** library — a framework for building Generative AI applications where context engineering is buildable at the control-flow level. Built on [footprintjs](https://github.com/footprintjs/footPrint) (the flowchart pattern for backend code).

## Core Thesis

**Building Generative AI applications is mostly *context engineering*** — deciding what content lands in which slot of the LLM call, when, and why. agentfootprint exposes this discipline through:

- **2 primitives** — `LLMCall`, `Agent` (= ReAct loop)
- **3 compositions + Loop** — `Sequence` · `Parallel` · `Conditional` · `Loop`
- **1 unifying injection primitive** — `Injection` with 4 typed sugar factories
- **1 memory factory** — `defineMemory({ type, strategy, store })`

Every named pattern (Reflexion, ToT, Swarm, ...) is a recipe over these. **Don't ship new classes per paper.**

## The Mental Model — Three Slots, Six Flavors

Every LLM call has three slots. Every "agent feature" is content flowing into one of them:

| LLM API field | What goes here |
|---|---|
| `system` prompt | Steering · Instruction text · Skill body · Fact data · formatted memory |
| `messages` array | Conversation history · RAG chunks · memory replay · injected instructions |
| `tools` array | Tool schemas (registered + Skill-attached) |

The flavors are how you *mark intent* — but they all reduce to one `Injection` primitive:

| Flavor | Trigger | Slots |
|---|---|---|
| **Skill** | LLM-activated (`read_skill`) | system-prompt + tools |
| **Steering** | Always-on | system-prompt |
| **Instruction** | Predicate (`activeWhen` / `on-tool-return`) | system-prompt or messages |
| **Fact** | Always-on (data) | system-prompt or messages |

## Mock-first development (RECOMMENDED workflow)

Build the entire app — agent, context engineering, tools, memory, RAG, MCP — against in-memory mocks first. Validate logic and patterns end-to-end with $0 API cost. Swap real infrastructure in, one boundary at a time, only after the flow is right.

```typescript
import { Agent, mock, InMemoryStore, mockEmbedder, defineTool } from 'agentfootprint';

// Mock LLM provider — no API key, deterministic reply.
const provider = mock({ reply: 'Refunds take 3 business days.' });

// Inline-mocked tool — no real backend yet.
const lookup = defineTool({
  name: 'lookup',
  description: '...',
  inputSchema: { type: 'object', properties: {} },
  execute: async () => 'mock data',
});

const agent = Agent.create({ provider, model: 'mock' }).tool(lookup).build();
await agent.run({ message: 'How long does a refund take?' });
```

| Boundary | Mock for development | Production swap |
|---|---|---|
| LLM provider | `mock({ reply })` · `mock({ replies })` for scripted ReAct | `anthropic()` · `openai()` · `bedrock()` · `ollama()` |
| Embedder | `mockEmbedder()` | OpenAI / Cohere / Bedrock embedder factory |
| Memory store | `InMemoryStore` | `RedisStore` (`agentfootprint/memory-redis`) · `AgentCoreStore` (`agentfootprint/memory-agentcore`) · DynamoDB / Postgres / Pinecone (planned) |
| MCP server | `mockMcpClient({ tools })` — in-memory, no SDK | `mcpClient({ transport })` to a real server |
| Tool execute | inline `async () => '...'` closure | real implementation |

When generating starter code for users, **default to the mock surface** unless they explicitly say they have a key / endpoint / store ready. Show real-provider code as the "swap" step, not the first step.

**Subpath imports** for memory adapters keep the main barrel small + tree-shaking clean:

```typescript
import { RedisStore } from 'agentfootprint/memory-redis';
import { AgentCoreStore } from 'agentfootprint/memory-agentcore';
```

Both lazy-require their SDK (`ioredis` / `@aws-sdk/client-bedrock-agent-runtime`) and accept `_client` for test injection.

**Multi-turn mock for tool-using ReAct:**

```typescript
const provider = mock({
  replies: [
    { toolCalls: [{ id: '1', name: 'lookup', args: { topic: 'refunds' } }] },
    { content: 'Refunds take 3 business days.' },
  ],
});
```

Each `complete()` consumes one reply in order. Exhaustion throws loud — misnumbered scripts fail tests instead of silently looping.

## Public API

### MCP — `mcpClient` (connect to MCP servers, register their tools)

```typescript
import { Agent, mcpClient } from 'agentfootprint';

const slack = await mcpClient({
  name: 'slack',
  transport: { transport: 'stdio', command: 'npx', args: ['@example/slack-mcp'] },
});

const agent = Agent.create({ provider })
  .tools(await slack.tools())  // pull ALL tools from the server in one call
  .build();

await agent.run({ message: '...' });
await slack.close();
```

Transports: `stdio` (local subprocess), `http` (Streamable HTTP). The
`@modelcontextprotocol/sdk` peer-dep is lazy-required — zero runtime
cost when MCP isn't used. Friendly install hint if missing.

`agent.tools(arr)` is the bulk-register companion to `agent.tool(t)`.
Pair with `await client.tools()` to register everything an MCP server

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [footprintjs/agentfootprint](https://github.com/footprintjs/agentfootprint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
