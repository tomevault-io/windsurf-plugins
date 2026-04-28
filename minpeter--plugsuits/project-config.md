---
trigger: always_on
description: **Updated:** 2026-03-09 KST
---

# PROJECT KNOWLEDGE BASE

**Updated:** 2026-03-09 KST
**Branch:** harness-decoupling

## OVERVIEW

`plugsuits` is a pnpm + Turborepo + TypeScript monorepo with four packages. The core agent harness (`@ai-sdk-tool/harness`) is model-agnostic and reusable. Terminal UI rendering lives in `@ai-sdk-tool/tui`, JSONL event streaming in `@ai-sdk-tool/headless`, and the full code-editing agent implementation in `@ai-sdk-tool/cea`.

## STRUCTURE

```text
plugsuits/
|- packages/
|  |- harness/          @ai-sdk-tool/harness — core loop, session, skills, TODO, commands
|  |  `- src/           Agent, CheckpointHistory, SessionManager, SkillsEngine, TodoContinuation
|  |- tui/              @ai-sdk-tool/tui — terminal UI components
|  |  |- src/           createAgentTUI, AssistantStreamView, BaseToolCallView, Spinner, colors
|  |  `- AGENTS.md      TUI package conventions
|  |- headless/         @ai-sdk-tool/headless — JSONL event streaming
|  |  |- src/           runHeadless, emitEvent, registerSignalHandlers, event types
|  |  `- AGENTS.md      Headless package conventions
|  `- cea/              @ai-sdk-tool/cea — code editing agent (uses all 3 packages)
|     |- src/
|     |  |- entrypoints/ CLI (interactive) + headless (JSONL) runtimes
|     |  |- tools/       edit_file, write_file, read_file, grep, glob, shell_execute
|     |  `- interaction/ stream rendering, spinner
|     `- benchmark/      Harbor terminal-bench adapter
|- scripts/             Benchmark and test automation
`- package.json         Workspace root — canonical scripts
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Interactive TUI entrypoint | `packages/tui/src/agent-tui.ts` | `createAgentTUI` — full terminal session loop |
| Headless JSONL runner | `packages/headless/src/runner.ts` | `runHeadless` — event-streaming loop |
| Core agent loop | `packages/harness/src/loop.ts` | `runAgentLoop` — model-agnostic iteration |
| Agent factory | `packages/harness/src/agent.ts` | `createAgent` — wraps Vercel AI SDK `streamText` |
| Message history | `packages/harness/src/checkpoint-history.ts` | `CheckpointHistory` — compaction + checkpointing |
| **Snapshot persistence** | `packages/harness/src/snapshot-store.ts` | `SnapshotStore` interface, `InMemorySnapshotStore` |
| **File-backed persistence** | `packages/harness/src/file-snapshot-store.ts` | `FileSnapshotStore` — wraps SessionStore, JSONL format |
| **Snapshot types** | `packages/harness/src/history-snapshot.ts` | `HistorySnapshot`, `SerializedMessage`, converters |
| Session management | `packages/harness/src/session.ts` | `SessionManager` — UUID-based session IDs |
| Skills loading | `packages/harness/src/skills.ts` | `SkillsEngine` — bundled/global/project skill discovery |
| TODO continuation | `packages/harness/src/todo-continuation.ts` | `TodoContinuation` — incomplete-task reminder loop |
| Command registry | `packages/harness/src/commands.ts` | `registerCommand`, `executeCommand`, `configureCommandRegistry` |
| Middleware chain | `packages/harness/src/middleware.ts` | `buildMiddlewareChain`, `MiddlewareConfig` |
| Stream rendering | `packages/tui/src/stream-handlers.ts` | `STREAM_HANDLERS` — per-part-type render dispatch |
| Tool call view | `packages/tui/src/tool-call-view.ts` | `BaseToolCallView`, `ToolRendererMap` |
| JSONL event types | `packages/headless/src/types.ts` | `TrajectoryEvent` union type |
| Signal handlers | `packages/headless/src/signals.ts` | `registerSignalHandlers` — SIGINT/SIGTERM/etc. |
| CEA tools | `packages/cea/src/tools/` | File edit, explore, shell execution tools |
| Benchmark adapter | `packages/cea/benchmark/AGENTS.md` | Trajectory conversion and validation constraints |
| **Runtime layer** | `packages/harness/src/runtime/` | `defineAgent`, `createAgentRuntime`, `AgentSession` — high-level DX layer |
| TUI session adapter | `packages/tui/src/session-tui.ts` | `runAgentSessionTUI` — connect AgentSession to TUI |
| Headless session adapter | `packages/headless/src/session-headless.ts` | `runAgentSessionHeadless` — connect AgentSession to headless |

## CODE MAP

| Symbol | Type | Location | Role |
|--------|------|----------|------|
| `createAgentTUI` | function | `packages/tui/src/agent-tui.ts` | Full interactive TUI session — input loop, stream rendering, command dispatch |
| `runHeadless` | function | `packages/headless/src/runner.ts` | JSONL event-streaming loop with optional TODO continuation |
| `runAgentLoop` | function | `packages/harness/src/loop.ts` | Model-agnostic agent iteration loop |
| `createAgent` | function | `packages/harness/src/agent.ts` | Wraps Vercel AI SDK `streamText` into an `Agent` |
| `CheckpointHistory` | class | `packages/harness/src/checkpoint-history.ts` | Conversation history with compaction and checkpointing |
| `SessionManager` | class | `packages/harness/src/session.ts` | UUID-based session ID lifecycle |
| `SkillsEngine` | class | `packages/harness/src/skills.ts` | Discovers and loads skills from bundled/global/project dirs |
| `TodoContinuation` | class | `packages/harness/src/todo-continuation.ts` | Reads todo files and generates reminder messages |
| `shouldContinueManualToolLoop` | fn | `packages/harness/src/tool-loop-control.ts` | Shared continuation gate — returns `true` for `"tool-calls"` finish reason |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [minpeter/plugsuits](https://github.com/minpeter/plugsuits) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
