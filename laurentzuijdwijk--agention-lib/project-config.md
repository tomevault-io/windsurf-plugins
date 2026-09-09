---
trigger: always_on
description: - `npm run build` - Build project (cleans dist folder then runs TypeScript compiler)
---

# Commands for agention-lib

## Build & Development
- `npm run build` - Build project (cleans dist folder then runs TypeScript compiler)
- `npm run watch` - TypeScript compiler in watch mode
- `npm run example` - Run example code
- `npm run docs` - Generate TypeDoc documentation
- `npm run jsdoc` - Generate JSDoc documentation

## Testing
- `npm test` - Run all tests
- `jest <path-to-file>` - Run specific test file (e.g., `jest lib/tools/Tool.spec.ts`)
- `npm run test:watch` - Run tests in watch mode

## Linting & Formatting
- `npm run lint` - Run ESLint on source files
- `npm run lint:fix` - Fix ESLint issues automatically
- `npm run format` - Format code with Prettier

## Code Style Guidelines
- **Types**: Use strict typing with explicit return types and interfaces
- **Classes**: PascalCase for classes/interfaces, camelCase for variables/methods
- **Imports**: Group imports by source
- **Documentation**: Write JSDoc comments for classes and methods
- **Error Handling**: Use try/catch with proper error propagation
- **Testing**: Write descriptive test blocks with proper mocking
- **Architecture**: Follow modular approach with base classes and inheritance

---

## Project Architecture

### Core Components

| Component | Location | Description |
|-----------|----------|-------------|
| `BaseAgent` | `lib/agents/BaseAgent.ts` | Abstract foundation for all LLM agents |
| `ClaudeAgent` | `lib/agents/anthropic/ClaudeAgent.ts` | Anthropic Claude implementation |
| `OpenAiAgent` | `lib/agents/openai/OpenAiAgent.ts` | OpenAI implementation (in progress) |
| `MistralAgent` | `lib/agents/mistral/MistralAgent.ts` | Mistral implementation |
| `Tool` | `lib/tools/Tool.ts` | Tool definition and execution |
| `MCPClient` | `lib/mcp/MCPClient.ts` | MCP server client — wraps MCP tools as `Tool` instances (optional peer: `@modelcontextprotocol/sdk`) |
| `History` | `lib/history/History.ts` | Conversation history management |
| `Team` | `lib/team/Team.ts` | Multi-agent coordination |

### Graph System (`lib/graph/`)

The graph module provides workflow orchestration patterns:

| Executor | Description |
|----------|-------------|
| `SequentialExecutor` | Chains agents, passing output to next |
| `ParallelExecutor` | Runs agents concurrently on same input |
| `Pipeline` | Chains any GraphNodes together |
| `MapExecutor` | Applies processor to each array item |
| `VotingSystem` | Judge selects best from multiple solutions |

**Key interfaces:**
- `GraphNode<TInput, TOutput>` - Base interface with `execute()`, `name`, and `nodeType`
- `GraphNodeType` - Type identifier: `"sequential" | "parallel" | "pipeline" | "map" | "voting" | "agent" | "custom"`

**Metrics & Observability:**
- `MetricsCollector` - Tracks timing, token usage, and pipeline structure
- `NodeExecutionMetrics` - Execution data for each node
- `PipelineMetrics` - Aggregate statistics for full pipeline runs
- Agents expose `lastTokenUsage` for metrics tracking

**Examples:** `examples/graph/` contains working examples for all executor types.

---

## Recent Changes

### Completed
- [x] Graph-based workflow system with 5 executor types
- [x] Metrics and observability (timing, token tracking, structure visualization)
- [x] Fixed filename typos (SequentialExecutor, MapExecutor)
- [x] Added `nodeType` property to all GraphNodes
- [x] Token usage tracking in ClaudeAgent (`lastTokenUsage` property)
- [x] Removed debug console.log statements from agents
- [x] Comprehensive tests for graph executors
- [x] Graph README and working examples
- [x] Moved `lastTokenUsage` to `BaseAgent` base class (all providers inherit it)
- [x] Fixed `BaseAgent.debug` default from `true` to `false`
- [x] Fixed `PlanStore.addStep()` step ID collision (now uses max existing index)
- [x] Fixed `PlanStore.updatePlanStatus()` to treat `skipped` steps as done
- [x] Fixed `PlanExecutor` with `stopOnFailure=true` to drain in-flight promises before throwing
- [x] Multi-provider multimodal support — `ImageUrlContent` / `ImageBase64Content` types, all 4 provider transformers updated, `BaseAgent.addImage()` helper, `execute()` accepts `string | MessageContent[]` on all agents. Fixed missing `system` param on Claude follow-up tool calls.
  - Future: per-image token estimation (currently flat 1000 tokens), `imageCompressionPlugin` for base64 history trimming
- [x] Added `LlamaCppAgent` (`lib/agents/llamacpp/`, vendor `"llamacpp"`, entry point `@agentionai/agents/llamacpp`) — talks to a local `llama-server` via its OpenAI-compatible `/v1/chat/completions` API using the `openai` SDK with a custom `baseURL`. New `chatCompletionsTransformer` in `lib/history/transformers.ts` handles the OpenAI Chat Completions message format (shared target for any OpenAI-compatible local server). Added `LlamaCppModel` type and `LlamaCppSpecificConfig`/`LlamaCppMeta`.
- [x] Added `listModels()` to both `OllamaAgent` (via `client.list()`) and `LlamaCppAgent` (via `client.models.list()` against `/v1/models`) for discovering locally-available models.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LaurentZuijdwijk/agention-lib](https://github.com/LaurentZuijdwijk/agention-lib) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
