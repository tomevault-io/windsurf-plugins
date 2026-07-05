---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is Omux

Omux is a chat-based meta-orchestrator that commands CLI coding agents (Claude Code, Codex, …) via tmux. It runs as a persistent HTTP + WebSocket server with a web chat UI. The MainAgent can hold natural conversations and autonomously execute complex development tasks by driving sub-agents inside tmux panes.

Core flow: **Chat message → MainAgent (IDLE ↔ EXECUTING state machine) → Streaming LLM → Tool execution in tmux → Response via WebSocket**

## Commands

```bash
npm run build          # tsc + copies src/agents/claude-code-skills → dist/agents/
npm run dev            # tsc --watch
npm test               # vitest run — all tests
npm run test:watch     # vitest — watch mode
npx vitest test/core/main-agent.test.ts            # run a single test file
npx vitest -t "name pattern"                       # run tests matching a name
npm run check          # biome check src/
npm run format         # biome format --write src/
npm start              # node --max-old-space-size=8192 dist/main.js (port 3120)
```

Subcommands of the `omux` binary (handled before server start): `config`, `doctor`, `init`, `remember`.

## Code Style

- **Formatter**: Biome — tabs, indent width 3, line width 120
- **Module system**: ESM (`"type": "module"` in package.json)
- **TypeScript**: strict mode, target ES2022, module Node16 — **use `.js` extension in relative imports**
- `noExplicitAny: off`, `noNonNullAssertion: off` — intentionally relaxed
- `useConst: error` — always prefer `const`

## Architecture

### Entry Point & CLI
- `src/main.ts` — orchestrates startup: (1) Bootstrap MemoryStore + EmbeddingProvider + memory sync + skill discovery/filter/registry + ConversationStore + CommandRegistry; (2) Restore conversation from SQLite if present; (3) Start Express+WS server; (4) SIGINT/SIGTERM → graceful shutdown.
- `src/cli.ts` — `parseCliArgs()` (--agent, --provider, --model, --base-url, --port, --cwd, …), `printHelp()`, `printVersion()`.

### MainAgent — `src/core/main-agent.ts`
Class **`MainAgent extends EventEmitter<MainAgentEvents>`** ([main-agent.ts:370](src/core/main-agent.ts:370)). Two-state machine: **IDLE** ↔ **EXECUTING**.

- **IDLE**: `handleMessage(content)` streams LLM response. Tool calls → EXECUTING. Pure text → stays IDLE.
- **EXECUTING**: self-loop. Between rounds: check `stopRequested`, drain `MessageQueue` (human messages received during execution), check context thresholds. Terminal tools (`mark_failed`, `escalate_to_human`) return to IDLE. Text-only LLM response also returns to IDLE.

All LLM calls go through `llmClient.stream()`; text deltas are broadcast to WebSocket clients in real time.

**Constructor options of note**: `createAgentSettleMs` (default 10_000 ms — wait after `create_agent` then capture the agent's initial pane for the LLM to see), `thinking` (ThinkingLevel), `learningPipeline`, `changeTracker`, `promptTracker`, `agentStore`.

**Events emitted**: `state_change`, `log`.

**Built-in tools exposed to the LLM** (declared inline in the same file):
- Agent interaction — `send_to_agent`, `respond_to_agent`, `interrupt_agent` (sends Esc + summary to the chat), `inspect_agent`, `wait_for_agents` (parks the loop until the next agent callback — terminal only when a wake-up is guaranteed; stops the LLM from polling), `create_agent`, `list_agents`, `kill_agent`
- Memory — `memory_search`, `memory_get`, `memory_edit` (modes: append/overwrite/replace/delete; `memory_write` is a backwards-compatible alias), `persistent_memory` (read/update global + project MEMORY.md)
- Discovery — `read_skill`, `exec_command` (read-only bash)
- Terminal — `mark_failed`, `escalate_to_human`

`send_to_agent` and `respond_to_agent` both require a `summary` parameter that's surfaced as `agent_update` in the chat UI.

### ContextManager — `src/core/context-manager.ts`
Class **`ContextManager`** ([context-manager.ts:42](src/core/context-manager.ts:42)). Modular system prompt with replaceable sections (`{{compressed_history}}`, `{{memory}}`, `{{agent_capabilities}}`).

Thresholds (defaults):
- `contextWindowLimit = 500_000`
- `flushThreshold = 0.6` — **Layer 2 / memory flush**: extract decisions, preferences, knowledge via `memory-flush.md` and persist to memory files.
- `compressionThreshold = 0.7` — **Layer 3 / compression**: compress history → reset context → re-inject POST_COMPACTION_CONTEXT.
- `toolResultRetention = 20`

Key methods: `getSystemPrompt()`, `getConversationId()`, `addMessage()`, `getMessages()`, `compress()`, `clear()`, `reloadPersistentMemory()`, and **`setCompactTuning({ tools, thinking })`** — wires the compaction LLM call into the same tool/thinking surface as regular turns so it rides the prompt cache. MainAgent calls this at startup; if not wired, compaction falls back to a separate billable completion.

Persistence: `addMessage()` auto-persists to SQLite when a `ConversationStore` is configured. `restore(store)` rebuilds state on server restart. `clear()` runs memory flush → wipes SQLite.

Uses hybrid token counting: last-known API count + pending character estimation.

### Persistence — `src/persistence/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Happenmass/omux](https://github.com/Happenmass/omux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-05 -->
