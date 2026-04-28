---
trigger: always_on
description: Damocles is a VS Code extension that integrates Claude AI as a coding assistant using the Claude Agent SDK. Webview-based chat interface with diff approval, tool visualization, session management, and MCP server support.
---

# CLAUDE.md

## Project Overview

Damocles is a VS Code extension that integrates Claude AI as a coding assistant using the Claude Agent SDK. Webview-based chat interface with diff approval, tool visualization, session management, and MCP server support.

## Development Commands

```bash
npm run build         # Build both extension and webview
npm run dev           # Watch mode for development
npm run typecheck     # Type checking
npm run lint          # Lint
npm test              # Run recall module tests (Vitest)
npm run package       # Package for distribution
```

**Testing:** Press F5 in VS Code to launch the Extension Development Host.

## Architecture

```
Extension Host (Node.js)                    Webview (Vue 3 + Pinia)
┌────────────────────────────┐              ┌──────────────────────────┐
│ ClaudeSession (SDK wrapper)│              │ App.vue + Pinia Stores   │
│ PermissionHandler          │◄─postMessage─│ message-handler/         │
│ ChatPanelProvider          │              │ Components               │
└────────────────────────────┘              └──────────────────────────┘
```

- **Extension:** esbuild → `dist/extension.js` (CJS). SDK, `sql.js-fts5`, `zod`, `web-tree-sitter` are external.
- **Webview:** Vite → `dist/webview/` (ESM). shadcn-vue + Tailwind + Shiki.
- **Type aliases:** `@shared/*` → `src/shared/*`, `@/*` → `src/webview/*`

### Key Modules

| Module | Purpose |
| --- | --- |
| `browser/` | Integrated browser via CDP: Chrome launch, screencast panel, element picker, 15 MCP tools |
| `claude-session/` | SDK integration: `query-manager.ts` (context usage, plugin reload), `system-prompt.ts` (custom system prompt builder), `streaming-manager/` (processor registry), tool/checkpoint/hook managers, `btw-handler.ts` (ephemeral side-questions) |
| `chat-panel/` | Webview management: panel, session, settings, message routing, history, workspace |
| `permission-handler/` | Tool permissions with domain-specific managers (approval, question, plan, skill, subagent, elicitation) |
| `memory/` | 5-tier persistent memory in WASM SQLite/FTS5. Two-phase lazy init. Pull-first catalog model with on-demand detail retrieval |
| `recall/` | Task-node-scoped context recall based on RLM paper. BM25 orientation → REPL sandbox retrieval. Per-node JSONL persistence |
| `voice/` | Speech-to-text via Whisper/Deepgram/Google Cloud |
| `team/` | Collaborative multi-agent teams: 2-5 specialists + lead via MessageBus + Scratchpad. 161 domain profiles from AgentLand |
| `compass/` | Workspace knowledge graph: tree-sitter AST extraction → graphology graph → Louvain clustering → 4 MCP tools |
| `session/` | JSONL session persistence with metadata cache for fast history listing |
| `auth/` | Damocles-owned OAuth + bundled-sidecar sign-in/out. Isolated from the standalone Claude Code CLI: own credentials file at `~/.damocles/auth/.credentials.json`, env-sanitized SDK spawns, dynamic mirror of `~/.claude/` config |

### Patterns

- **Facade + DI:** Each module has an `index.ts` facade. Managers receive dependencies through constructor.
- **Two-phase lazy init:** Constructor reads config only; `ensureInitialized()` defers heavy work (WASM, DB, graph) to first access. Used by Memory, Compass.
- **Message routing:** Both sides use domain-handler registries: `message-router/handlers/` (extension) and `message-handler/handlers/` (webview).

## Memory Module

WASM SQLite with FTS5 at `~/.damocles/memory.db`. Lazy ESM `import()` for MCP server + Zod schemas. Pull-first catalog (~300-800 tokens per prompt); Claude calls `get_memory_details` on demand. Observation staleness tracked via `FileChangeTracker`.

## Team Module

2-5 specialist agents collaborate via in-process `MessageBus` + `Scratchpad`, coordinated by a lead agent. Each agent runs as an independent SDK `query()` with a scoped MCP server. Disabled by default (`damocles.team.enabled`).

**Key design decisions:**
- **Deliberative collaboration:** Lead facilitates (no independent research); specialists must read peer scratchpad sections and cross-reference before reporting
- **Two MCP server factories:** `createTeamMainMcpServer()` (3 tools for main session) and `createTeamAgentMcpServer()` (8 tools per agent, lead-only tools gated by role)
- **Event-driven keep-alive:** Lead blocks on bus notifications, wakes on specialist completion (no polling)
- **Synthesis guard:** `team_synthesize_result` rejects if any specialist still running — lead must wait or cancel
- **Review gate:** `team_approve_specialist` and `team_request_revision` mechanically blocked until all specialists are settled (dynamic `isReviewRoundReady()` check). `approveSpecialist()` also rejects if specialist has a pending revision (`pendingReportComplete` guard). Keep-alive message shows count-only for awaiting-review to suppress premature attempts
- **Lead broadcast filter:** `shouldDeliverMessage: (msg) => msg.to !== null` — lead only wakes on direct messages (`[REVIEW ROUND READY]`, specialist completion/failure, direct questions), not scratchpad broadcasts. Specialists handle cross-review autonomously via task prompts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AizenvoltPrime/damocles](https://github.com/AizenvoltPrime/damocles) — distributed by [TomeVault](https://tomevault.io/claim/AizenvoltPrime).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
