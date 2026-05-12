---
trigger: always_on
description: This document defines **mandatory rules** for any automated agent (LLM, code generator, bot) operating in this repository.
---

# agents.md — Autonomous Agent Execution Contract (v3)

This document defines **mandatory rules** for any automated agent (LLM, code generator, bot) operating in this repository.

**Violation of these rules produces incorrect output.**

***

## 0. SOURCE OF TRUTH HIERARCHY

```
PRD.md > agents.md > existing code > your assumptions
```

**Before writing, modifying, or suggesting ANY code:**

1. **Read `PRD.md`** — Product and architecture specification
2. **Read `agents.md`** — Runtime and implementation contract
3. **Cross-reference the Vercel AI SDK docs** — Authoritative API reference

**Rules:**
- Do **NOT** reimplement functionality that Vercel AI SDK provides (see Section 2)
- Do **NOT** invent custom streaming protocols (use AI SDK UI message stream via `toUIMessageStreamResponse()`)
- Do **NOT** create custom message types (use AI SDK `UIMessage`/`ModelMessage`)
- Do **NOT** build custom LLM↔tool loops (use `streamText()`)
- Cite specific doc sections when referencing requirements

**If conflict exists:** Quote the exact text and stop.

***

## 1. RUNTIME CONTRACT: Bun-Only Architecture

**Bun is the exclusive runtime.** Node.js compatibility is explicitly deprioritized.

### Mandatory Rules
```
✅ Use: Bun.serve(), Bun.file(), Bun.spawn()
✅ Use: bun:sqlite (local file mode, sync API)
❌ Never: Node fs, child_process, Express, Fastify
✅ Default: bun --watch engine/index.ts serve
❌ Never: npm start, nodemon, ts-node
```

***

## 2. VERCEL AI SDK CONTRACT (Critical)
 
 PocketAgents is **infrastructure for Vercel AI SDK**, not a competing framework.
 
 ### What Vercel AI SDK Owns (NEVER Reimplement)
 ```
 streamText()                    — Agentic loop (LLM ↔ tool execution ↔ streaming)
 toUIMessageStreamResponse()     — Standardized UI message stream response
 useChat()                       — React hook: message state, streaming, tool approval
 UIToolInvocation               — Client-side tool call state
 ModelMessage / UIMessage        — Standard message formats
 AI SDK UI Message Stream        — The wire format
 ```

### What PocketAgents Owns
```
resolveAgent()                  — DB lookup: agent → model → provider key → tools → RAG
Tool HTTP proxy                 — executeHttpTool() with provider keys injection
Tool internal handlers          — search_memory
RAG pipeline                    — chunk → embed → store → search → inject
Event logging                   — Store run events to events table
Auth middleware                 — Admin/client key validation (Bearer token)
CRUD API routes                 — agents, models, provider keys, tools, collections
Admin console                   — React SPA using useChat() + TanStack Query
Single binary                   — bun build --compile
```

### The Chat Pipeline (Entire Server-Side Streaming)
```ts
const resolved = await resolveAgent(agentId);     // 1. Resolve from DB (agent, model, provider key, tools)
const result = streamText({ ...resolved });        // 2. Vercel AI SDK does everything
return result.toUIMessageStreamResponse();         // 3. Return Response
```

Any code that reimplements what `streamText()` already does is **wrong**.

***

## 3. LANGUAGE: TypeScript Everywhere (Strict)

**All code must be TypeScript.** `.js` files are prohibited.

### Type Requirements
```
✅ Explicit types for public APIs
✅ Use AI SDK types: UIMessage, ModelMessage, UIToolInvocation
✅ Interfaces for module boundaries
✅ No `any` unless documented exception
❌ Custom ChatMessage / ChatResponse types (use AI SDK's)
❌ Implicit `any` → wrong
```

### File Structure
```
engine/
├── index.ts              # Single entrypoint (bun build --compile target)
├── agent-engine.ts       # resolveAdapter() — provider adapter factory
├── config.ts             # CLI flags → runtime config
├── api/                  # Service modules (CRUD + business logic)
│   ├── agents.ts
│   ├── chat.ts           # handleChat() — resolveAgent + streamText() + toDSR
│   ├── collections.ts
│   ├── keys.ts
│   ├── models.ts
│   ├── provider-keys.ts
│   ├── runs.ts           # handleRun() — logAndForward() async generator
│   └── tools.ts
├── server/
│   ├── routes.ts         # Bun.serve route handlers
│   ├── auth.ts           # Auth middleware
│   └── helpers.ts        # Response helpers
├── runtime/
│   ├── resolve.ts        # resolveAgent() — shared resolution pipeline
│   ├── tools.ts          # rowToServerTool() — DB → AI SDK tool definition
│   ├── rag.ts            # buildRagContext() — passive RAG injection
│   ├── embeddings.ts     # Embedding client
│   └── ingestion.ts      # Document chunking pipeline
├── db/
│   └── index.ts          # bun:sqlite connection + migrations
├── shared/
│   └── types.ts          # DB row types (AgentRow, ModelRow, ToolRow, etc.)
└── utils/
    ├── logger.ts
    ├── crypto.ts
    ├── provider-keys.ts
    └── validation.ts     # JSON Schema validation for tool args
```

***

## 4. DEPENDENCY POLICY: Approved Only

### Approved
```
@ai-sdk/openai, @ai-sdk/anthropic, etc. (Model Providers)
 ai, @ai-sdk/react                           (AI SDK Core + React)
@tanstack/react-query, @tanstack/react-router         (UI data + routing)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [treyorr/pocket-agents](https://github.com/treyorr/pocket-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
