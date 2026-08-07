---
trigger: always_on
description: This file contains key information about the project for AI agents to study and align with when working on Superagent.
---

# Project Specifications (agents.md) 

This file contains key information about the project for AI agents to study and align with when working on Superagent.

## Project Overview
- **Name**: Superagent
- **Description**: An interactive, terminal-based AI coding assistant featuring a cyberpunk style terminal UI, context token tracking, and a 3-tier multi-agent orchestration system (Master Agent → Superagent → Subagent).
- **Technology Stack**: Node.js, TypeScript, React, Ink (Terminal UI Components), Vercel AI SDK, Execa, Vitest
- **Desktop Client Integration (t-line)**: Superagent integrates with [t-line](file:///D:/backup%20from%20pc%20asus/Documents%20Development/t-line), which is the desktop version of the superagent CLI. This is supported via a client/server bridge mode (`superagent --server [port] --client-mode tline`).

## 3-Tier Multi-Agent Architecture

Superagent supports a full 3-tier agent hierarchy activated via `superagent --multi`:

```
Master Agent  (orchestrator)
  └── Superagent  (per-feature, isolated in git worktree)
        └── Subagent  (atomic ops, ephemeral)
```

### Tier Responsibilities
| Tier | Role | Toolset | Isolation |
|------|------|---------|-----------|
| **Master Agent** | Orchestration, planning, result merging | `invokeSuperagentTool`, `awaitSuperagentsTool`, `mergeSuperagentsTool`, `manageSuperagentsTool`, `manageSubagentsTool`, `gitWorktreeTool` | Main repo |
| **Superagent** | Feature-level development | Shell + File tools + `invokeSubagentTool`, `manageSubagentsTool`, `gitWorktreeTool` | Isolated git worktree (`~/.superagent-r/worktrees/<name>`) |
| **Subagent** | Ephemeral subagent types (e.g. `researcher`, `coder`, `reviewer`, `software-tester`, `chrome-agent`). Note that Chrome/browser control is isolated to `chrome-agent`. | Tools allowed per subagent type | Ephemeral, within parent worktree |

### Key Files
- `src/core/masterAgent.ts` — Master Agent entry point and orchestrator logic
- `src/core/tools/types.ts` — Shared types: `AgentTier`, `SubagentInstance`, `ToolSet`
- `src/core/tools/toolsets.ts` — ToolSet definitions keyed per tier (`masterToolset`, `superagentToolsets`, `subagentToolsets`)
- `src/core/tools/prompts.ts` — System prompts per tier with dynamic context injection
- `src/core/tools/state.ts` — Shared subagent registry, instances map, event emitters
- `src/core/tools/superagentTools.ts` — Master tier tools: invoke/list/merge/manage Superagents
- `src/core/tools/subagentTools.ts` — Superagent tier tools: spawn ephemeral Subagents
- `src/core/context/ContextManager.ts` — Central orchestrator for context window management (state machine, strategy selection, recovery)
- `src/core/context/TokenTracker.ts` — Model-specific token counting via tiktoken (includes tool calls/results)
- `src/core/context/CompactionStrategy.ts` — Pluggable strategy interface for compaction algorithms
- `src/core/context/strategies/SummarizationStrategy.ts` — LLM-based summarization (with heuristic fallback)
- `src/core/context/strategies/PruningStrategy.ts` — Emergency pruning with summary preservation (never silent context loss)
- `src/core/context/strategies/PinningStrategy.ts` — Preserve critical pinned messages during compaction
- `src/core/context/SemanticAnalyzer.ts` — Topic boundary detection, importance scoring, key point extraction
- `chrome-extension-remote/` — Standalone lightweight Chrome Extension (Manifest V3) for remote-controlling any Chrome profile/browser from Superagent CLI via serverless WebSocket bridge (port 9223). No sidepanel GUI; minimal footprint, multi-profile capable.
- `chrome-extension/` — Full Superagent AI assistant Chrome Extension (Manifest V3) with sidepanel GUI for browser control, tab/DOM management, text/markdown extraction, console/network logs, storage/cookies management, device emulation, and more. Installed directly in target browser it controls.
- `src/core/tools/remoteChromeBridge.ts` — Serverless WebSocket server (port 9223) auto-initialized on-demand by Superagent CLI; communication backbone for `chrome-extension-remote/`.

## Coding Guidelines & Constraints
- **Language — English Only**: All user-facing text strings, UI labels, log messages, comments, variable names, documentation, and any other text content MUST be written in English. No exceptions.
- **Shell Commands**: On Windows, the actual shell is auto-detected (Git Bash is preferred over PowerShell). If using PowerShell, use `;` to separate commands instead of `&&`. Git Bash supports `&&` normally. The system prompt reports the detected shell accurately.
- **Strict Naming Rules**: Do NOT mention proprietary brand names like "Claude Code" or generic "CLI" terms in user-facing documentation or UI descriptions. Refer to the project as a terminal-based AI coding assistant.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RudyCity/superagent](https://github.com/RudyCity/superagent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
