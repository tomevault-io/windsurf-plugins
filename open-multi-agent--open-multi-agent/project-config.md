---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run build          # Compile TypeScript (src/ → dist/)
npm run dev            # Watch mode compilation
npm run lint           # Type-check only (tsc --noEmit)
npm test               # Run all tests (vitest run)
npm run test:watch     # Vitest watch mode
npm run test:coverage  # Vitest with v8 coverage
npm run test:e2e       # E2E suite (requires RUN_E2E=1, real API keys)
node dist/cli/oma.js help   # After build: shell/CI CLI (`oma` when installed via npm bin)
```

Tests live in `tests/` (vitest); E2E suite under `tests/e2e/`. Examples in `examples/` are standalone scripts requiring API keys (`ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, etc.) and are organized by intent: `basics/`, `cookbook/`, `patterns/`, `providers/`, `integrations/`, `production/`, with shared `fixtures/`. CLI usage and JSON schemas: `docs/cli.md`.

## Architecture

ES module TypeScript framework for multi-agent orchestration. Three runtime dependencies: `@anthropic-ai/sdk`, `openai`, `zod`. Optional peer deps `@aws-sdk/client-bedrock-runtime` (Bedrock), `@google/genai` (Gemini), and `@modelcontextprotocol/sdk` (MCP) are loaded lazily so users only install what they use; the three-dependency promise covers `dependencies` only.

### Core Execution Flow

**`OpenMultiAgent`** (`src/orchestrator/orchestrator.ts`) is the top-level public API with three execution modes:

1. **`runAgent(config, prompt)`** — single agent, one-shot
2. **`runTeam(team, goal)`** — automatic orchestration: a temporary "coordinator" agent decomposes the goal into a task DAG via LLM call, then tasks execute in dependency order
3. **`runTasks(team, tasks)`** — explicit task pipeline with user-defined dependencies

### The Coordinator Pattern (runTeam)

This is the framework's key feature. When `runTeam()` is called:
1. A coordinator agent receives the goal + agent roster and produces a JSON task array (title, description, assignee, dependsOn)
2. `TaskQueue` resolves dependencies topologically — independent tasks run in parallel, dependent tasks wait
3. `Scheduler` auto-assigns any unassigned tasks (strategies: `dependency-first` default, `round-robin`, `least-busy`, `capability-match`)
4. Each task result is written to `SharedMemory` so subsequent agents see prior results
5. The coordinator synthesizes all task results into a final output

### Layer Map

| Layer | Files | Responsibility |
|-------|-------|----------------|
| Orchestrator | `orchestrator/orchestrator.ts`, `orchestrator/scheduler.ts` | Top-level API, task decomposition, coordinator pattern |
| Team | `team/team.ts`, `team/messaging.ts` | Agent roster, MessageBus (point-to-point + broadcast), SharedMemory binding |
| Agent | `agent/agent.ts`, `agent/runner.ts`, `agent/pool.ts`, `agent/structured-output.ts`, `agent/loop-detector.ts` | Agent lifecycle (idle→running→completed/error), conversation loop, concurrency pool with Semaphore, structured output validation, sliding-window loop detection |
| Task | `task/queue.ts`, `task/task.ts` | Dependency-aware queue, auto-unblock on completion, cascade failure to dependents |
| Tool | `tool/framework.ts`, `tool/executor.ts`, `tool/mcp.ts`, `tool/text-tool-extractor.ts`, `tool/built-in/` | `defineTool()` with Zod schemas, ToolRegistry, parallel batch execution with concurrency semaphore, MCP integration, fallback text-format tool-call extraction for local models |
| LLM | `llm/adapter.ts` + per-provider files (`anthropic`, `openai`, `azure-openai`, `bedrock`, `gemini`, `grok`, `deepseek`, `minimax`, `qiniu`, `copilot`) + `openai-common.ts` | `LLMAdapter` interface (`chat` + `stream`); async `createAdapter()` factory imports the chosen adapter lazily so unused SDKs aren't loaded; `baseURL` parameter targets OpenAI-compatible servers (Ollama, vLLM, LM Studio) |
| Memory | `memory/shared.ts`, `memory/store.ts` | Namespaced key-value store (`agentName/key`), markdown summary injection into prompts. Custom backends via `TeamConfig.sharedMemoryStore` (any `MemoryStore` impl); `sharedMemory: true` uses the default in-process store |
| Dashboard | `dashboard/render-team-run-dashboard.ts`, `dashboard/layout-tasks.ts` | Pure HTML renderer for the post-run team task DAG (no I/O) |
| CLI | `cli/oma.ts` | Shell/CI entry; built to `dist/cli/oma.js` and exposed as the `oma` npm bin |
| Utils | `utils/semaphore.ts`, `utils/tokens.ts`, `utils/keywords.ts`, `utils/trace.ts` | Concurrency primitive, token accounting, keyword helpers, trace plumbing |
| Errors | `errors.ts` | Shared error types |
| Types | `types.ts` | All interfaces in one file to avoid circular deps |
| Exports | `index.ts` (root, `'@open-multi-agent/core'`), `mcp.ts` (subpath, `'@open-multi-agent/core/mcp'`) | Public API surface; MCP integration is a separate entry point so non-MCP users don't pay the import cost |

### Agent Conversation Loop (AgentRunner)

`AgentRunner.run()`: send messages → extract tool-use blocks → execute tools in parallel batch → append results → loop until `end_turn` or `maxTurns` exhausted. Accumulates `TokenUsage` across all turns.

### Context Compaction


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-multi-agent/open-multi-agent](https://github.com/open-multi-agent/open-multi-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
