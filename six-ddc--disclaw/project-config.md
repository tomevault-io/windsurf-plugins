---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Disclaw is a Discord bot that bridges Discord with Claude Code via the Agent SDK, enabling thread-based AI conversations with multimodal support. Architecture: `Discord Gateway → Bot → In-process JobRunner → Claude Agent SDK`. Single-process, no external dependencies beyond Discord. Runs locally with no exposed ports (outbound Discord gateway only).

## Commands

```bash
# Install dependencies
bun install

# Service management (auto-detects Linux systemd / macOS launchd)
make install       # Register as service (auto-detects project dir and bun path)
make start         # Start the service
make stop          # Stop the service
make restart       # Restart the service
make status        # Show service status
make logs          # Follow service logs
make deploy        # Type-check + restart
make uninstall     # Remove service + unlink skills
make link-skills   # Symlink project skills to ~/.claude/skills/
make unlink-skills # Remove skill symlinks from ~/.claude/skills/

# Development
make dev           # Start with hot reload (bun --watch, not as service)
make typecheck     # Type checking (bunx tsc --noEmit)
```

There are no tests or linting configured.

## Architecture

**Request flow:** User @mentions bot in Discord → `bot.ts` creates a thread, stores thread→session mapping in SQLite (session ID empty), submits job to `runner.ts` → runner calls `claude-client.ts` which uses `@anthropic-ai/claude-agent-sdk` `query()` to get an async iterator of SDKMessages → SDK auto-generates session ID, saved to DB on init message → messages are converted via `message-converter.ts` and rendered to Discord via `discord-sender.ts`. Follow-up messages in the thread use `resume: sessionId` to continue the session.

**Session ID lifecycle:** The bot never generates session UUIDs — the SDK auto-generates them. On each query, the SDK sends an init message (`type: 'system', subtype: 'init'`) containing `session_id`. The runner compares this with the stored ID; if different (new session), it saves the SDK ID to DB and sends a "New session" / "Forked session" notification embed. Ephemeral sessions (`persistSession: false`, e.g. cron) skip this.

**Working directory resolution (fallback chain):** `[/path]` message prefix override → `mapping.working_dir` (thread-level) → channel config → `CLAUDE_WORKING_DIR` env → `~/.disclaw`. Centralized in `working-dir.ts`. Thread-level override is set via `/disclaw cd` in a thread, which also clears the session (new directory = fresh session).

**Key source files:**
- `src/bot.ts` — Discord event routing (MessageCreate, MessageReactionAdd/Remove, interaction dispatch). No command handler logic — delegates to `interactions.ts`
- `src/interactions.ts` — All `/disclaw` slash command handlers. Thread-only commands use `requireThreadSession()` guard. Config modal (model/permission/display) uses `LabelBuilder` + `ModalBuilder`
- `src/dir-picker.ts` — Interactive button-based directory browser for Discord. Navigation: subdirectory buttons (paginated), Up/Prev/Next, Select/Cancel. 2-minute timeout
- `src/runner.ts` — In-process job runner with concurrency control (semaphore, default 10), per-thread job serialization with message batching/merging, retry with exponential backoff. Tracks active `Query` objects per thread for interrupt support. Detects session changes from SDK init messages. Stall detection with auto-resume for persistent sessions
- `src/claude-client.ts` — SDK wrapper; calls `query()` and streams SDKMessages via `onMessage` callback. Supports `model`, `forkSession`, `resumeSessionAt`, `canUseTool`, `permissionMode`, `persistSession` options
- `src/user-input.ts` — Handles SDK `canUseTool` callback via Discord interactive components. Manages AskUserQuestion (buttons/selects/modals) and tool approval UI (Allow/Deny/Always Allow). Per-thread auto-approved tools. 5-minute timeout
- `src/attachment-handler.ts` — Multimodal content extraction: images (PNG/JPEG/GIF/WebP, max 20MB, max 5), PDFs (max 20MB), text files (max 100KB), and reply references. Converts Discord attachments to Claude API content blocks. Type definitions for `ContentBlock` and `MultimodalPrompt`
- `src/message-converter.ts` — Transforms raw SDKMessages into normalized `ClaudeMessage` objects (text, tool_use, tool_result, thinking, system, permission_denied, task_started, task_notification, task_progress, tool_progress, tool_summary, other)
- `src/discord-sender.ts` — Renders `ClaudeMessage` objects as Discord embeds/messages with rich formatting. Delegates tool embed construction to `tool-embeds.ts`. Handles completion stats, session change notifications, and auto-deleting status messages
- `src/history.ts` — Paginated session history viewer. Uses SDK `getSessionMessages()`, renders as Discord embed with ◀/▶ navigation buttons. Used by fork, resume, and rewind
- `src/discord.ts` — Discord REST API helpers (send, edit, typing indicators), markdown splitting with table flattening

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [six-ddc/disclaw](https://github.com/six-ddc/disclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
