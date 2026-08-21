---
trigger: always_on
description: Copilot agent for [OpenBB Workspace](https://pro.openbb.co). Hono + Bun + Vercel AI SDK + MCP.
---

Copilot agent for [OpenBB Workspace](https://pro.openbb.co). Hono + Bun + Vercel AI SDK + MCP.

For architecture, tool inventory, file layout, env vars, request flow, the MCP and citation protocols, and the test strategy, read [README.md](./README.md). This file is rules, conventions, and gotchas only.

## Commands

```bash
bun run dev:agent     # agent on :7777
bun run dev:mcp       # MCP server on :8787
bun run typecheck     # bunx tsc --noEmit — must be zero errors
bun run test          # bun test tests/ — must be all green before merge
```

Prefer `bun`

## Hard rules — thin harness

The agent owns: protocol round-trips, widget tier/parse, citation aggregation, system prompt, **state-bound capabilities** (SQL family on `pendingTables` + `artifactQueue`, workspace bridge ops on the vendored bridge contract). Stateless / heavy / cold-start capabilities live in the MCP server. New capability needs in-process state the workspace can't reach → it's an agent tool; otherwise add it to `mcp-server/`.

- **No content-based routing in the harness.** No `isStructured()`, no `if data looks like X then path A else path B`. Surface tools, let the model pick. Generic dispatch by tool source (in-process vs SSE round-trip vs MCP) is structural, not content-based.
- **Never open MCP connections from the agent.** The workspace owns MCP. The agent emits `executeAgentTool` SSE for external MCP tools, native `copilotFunctionCall { function: <command> }` for the 16 vendored workspace bridge ops, or runs locally (`search_widgets`, SQL family).
- **One `streamText` per iteration.** The outer `MAX_LOOPS=3` exists only for cache-resolved widget re-fetches without an HTTP round-trip — not for orchestration. (`streamText`, not `generateText`: the loop consumes `fullStream` so in-process tool chains stream progress live; a stalled call is bounded by `LLM_STREAM_TIMEOUT_MS`.)
- **Tools have `execute` only when the work is local + cheap, *or* state-bound to in-process data.** Today that's `search_widgets` (closure on `tieredWidgets` — primary/secondary/extra preserved for ranking) and the SQL family (closure on `pendingTables` + `artifactQueue`). Everything else (widget data, skill, workspace bridge ops, MCP-registered) has no `execute`.
- **Prepare, don't prescribe.** Ship widget rows into `pendingTables`; the model decides whether to call SQL, compute, or answer directly.

The only acceptable harness branching is `lastMessage.role === "tool"` dispatched by `toolMsg.function` (`get_widget_data` default, `get_skill_content`, `execute_agent_tool`).

## TypeScript

- `strict: true`, zero errors at all times. CI gates on this.
- No `any` unless unavoidable, and justify with a comment. Prefer `unknown` + type guards.
- `interface` for object shapes, `type` for unions/intersections.
- Wire types live in `src/protocol/types.ts`; MCP-result types in `src/mcp/results.ts`; ContentItem helpers in `mcp-server/src/lib/typed.ts`.

## Zod is the source of truth

- Every tool input is a Zod schema. Validate with `.parse()` — **never `as`**.
- Use `.describe()` on every field; the description is what the LLM sees.
- The MCP schema layer (`src/mcp/schema.ts`) handles `enum`, `anyOf`, `oneOf`, `null`, nested `object`, `array.items`, `nullable`, `const`. Extend it rather than collapsing into `z.unknown()`.

## Vercel AI SDK patterns

```ts
// Auto-execute tool — local + cheap, or state-bound to in-process data
const t = tool({ description, inputSchema, execute: async (args) => { ... } });

// Round-trip tool: omit execute → loop stops, harness emits SSE
const rt = tool({ description, inputSchema });
```

`stopWhen` includes one `hasToolCall(name)` per round-trip tool — `get_widget_data`, `get_skill_content`, every workspace bridge command name (when `generativeUiEnabled`), plus `mcpToolsResult.stopCondition` for external MCP tools, plus `stepCountIs(15)`.

Consume `result.fullStream` and emit live as parts arrive (`src/agent/loop.ts`):
- `text-delta` → buffer; flush+`messageChunk` on `text-end` (whole-segment, so `stripPlaceholderTags` can't be defeated by a tag split across deltas — token-level text streaming is intentionally deferred)
- `tool-call` → `reasoningStep` per call, carrying `{ tool_name, input }` (the eval trace runner reads this to attribute in-process calls)
- drain `artifactQueue` after every part (SQL family pushes synchronously from `execute`; MCP results push from `processMcpResult` on the next re-POST) — artifacts land right after the tool that produced them
- `abort`/`error` (or a thrown stream error) → ERROR `reasoningStep` + return
- after the stream drains, `await result.steps` / `result.totalUsage` / `result.text` for the round-trip dispatch (reads `lastStep` / `finalText`)

Tools never yield SSE directly. The `artifactQueue` is the single ordered side-channel.

## Decoration (`x-agentrita-*`)

Some data must ride along with an MCP call without the LLM seeing it. Two capabilities use this path: `execute_code` (`x-agentrita-conversation-id`, `x-agentrita-tables`) and the document-RAG tools `query_documents` / `list_documents` (`x-agentrita-conversation-id`, `x-agentrita-documents`). Three rules:

1. Inject into `parameters` in `src/agent/loop.ts` right before yielding `executeAgentTool`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenBB-finance/agent-rita](https://github.com/OpenBB-finance/agent-rita) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
