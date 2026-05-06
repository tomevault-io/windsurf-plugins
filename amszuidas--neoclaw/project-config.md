---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
bun install          # Install dependencies
bun start            # Start daemon (self-daemonizes to background, logs to ~/.neoclaw/logs/neoclaw.log)
bun onboard          # Generate ~/.neoclaw/config.json config template
bun run dev          # Run with --watch (auto-restart on file changes)
bun run typecheck    # Type-check without emitting (bun run --bun tsc --noEmit)
bun run lint         # Lint with ESLint + Prettier
```

### CLI (citty)

Entry point: `src/index.ts` → `src/cli/index.ts`. Uses [citty](https://github.com/unjs/citty) for CLI framework. Subcommands are lazily loaded from `src/cli/commands/`.

```bash
neoclaw start        # Start daemon (alias: bun start)
neoclaw stop         # Stop running daemon
neoclaw onboard      # Generate config template
neoclaw cron <sub>   # Cron job management (create, list, delete, update)
```

## Architecture

Gateway pattern. `Dispatcher` (`dispatcher.ts`) routes messages from **Gateways** (I/O adapters) to **Agents** (AI backends).

```
Gateway.start(dispatcher.handle)
  → InboundMessage + ReplyFn + StreamHandler → Dispatcher → Agent.stream() → AgentStreamEvent*
                                                                            ↓
                                                             streamHandler(stream) → Gateway renders
```

**Message flow:**
1. Gateway receives a raw platform event, parses it into `InboundMessage`, creates a `reply` closure and a `streamHandler` closure with protocol context bound
2. Gateway calls `dispatcher.handle(msg, reply, streamHandler)`
3. Dispatcher acquires a per-conversation `SerialQueue` lock to prevent race conditions
4. Checks for slash commands (`/clear`, `/new`, `/restart`, `/status`, `/help`) — always use non-streaming `reply()`
5. If `streamHandler` is provided and agent has `stream()`, uses streaming path; otherwise falls back to `agent.run()` + `reply()`
6. Gateway's `streamHandler` renders content progressively as events arrive

**Key interfaces** (in `*/types.ts`):
- `Agent`: `run()`, `stream?()`, `healthCheck()`, `clearConversation()`, `dispose()` — AI backend
- `Gateway`: `start(handler)`, `stop()`, `send()` — messaging platform adapter
- `ReplyFn`: `(response: RunResponse) => Promise<void>` — for slash commands and non-streaming fallback
- `StreamHandler`: `(stream: AsyncIterable<AgentStreamEvent>) => Promise<void>` — for progressive rendering
- `MessageHandler`: `(msg, reply, streamHandler?) => Promise<void>` — called by Gateway for each message
- `AgentStreamEvent`: `{ type: 'thinking_delta', text }` | `{ type: 'text_delta', text }` | `{ type: 'tool_use', name, input }` | `{ type: 'ask_questions', questions, conversationId }` | `{ type: 'done', response: RunResponse }`
- `InboundMessage`: `id`, `text`, `chatId`, `threadRootId?`, `authorId`, `authorName?`, `gatewayKind`, `chatType?` (`'private'` | `'group'`), `attachments?: Attachment[]`, `meta?`
- `RunResponse`: `text`, `thinking?`, `sessionId?`, `costUsd?`, `inputTokens?`, `outputTokens?`, `elapsedMs?`, `model?`

**Agents**: `ClaudeCodeAgent` uses Claude Code CLI via long-running subprocess with bidirectional JSONL streaming. Maintains one subprocess per `conversationId` (pooled, reaped after 10 min idle). After idle reap, session IDs are persisted in `~/.neoclaw/cache/sessions.json` so the next request **resumes** the same Claude session (`--resume <sessionId>`). Each conversation runs in its own workspace directory `~/.neoclaw/workspaces/<conversationId>` (`:` replaced with `_`); the directory is created on first use. Default model: `claude-sonnet-4-6`. Passes `NEOCLAW_CHAT_ID` and `NEOCLAW_GATEWAY_KIND` env vars to the subprocess for cron/context routing. Disallows `CronCreate`, `CronDelete`, `CronList` tools (handled via CLI instead).

The `stream()` method on `ClaudeCodeAgent` yields `AgentStreamEvent`s: `thinking_delta` and `text_delta` are emitted for each JSONL `content_block_delta`; `tool_use` events pass through tool call info; `ask_questions` events surface interactive permission/question prompts; a final `done` event carries the full `RunResponse` (including stats).

**File access security** (`agents/file-blocked-agent.ts`, `utils/file-guard.ts`): `createFileBlockedAgent()` wraps any Agent to enforce file access policies:
- **File blacklist**: Blocks reading/writing files matching configurable glob patterns (supports `*`, `**`, `?`, tilde expansion). Default blacklist covers `~/.claude/**`, `~/.config/claude/**`, `/etc/shadow`, `/etc/passwd`, `**/.env`, `**/credentials.json`, `**/secrets/**`, `~/.neoclaw/config.json`, `~/.neoclaw/config.json.backup`.
- **Group chat write restrictions**: For `chatType === 'group'`, blocks write operations (`rm`, `mv`, `cp`, `touch`, `mkdir`, `chmod`, `chown`, `ln`, `tee`, `>`, `>>`, Write/Edit/NotebookEdit tools) targeting paths outside the conversation's workspace directory.
- Intercepts `tool_use` stream events; in non-streaming mode returns error message instead of executing.

**Gateways**:

*FeishuGateway* (Feishu/Lark WebSocket). Handles:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amszuidas/neoclaw](https://github.com/amszuidas/neoclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
