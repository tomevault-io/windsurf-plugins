---
trigger: always_on
description: This is a multi-platform bot that lets users interact with Claude Code through chat platforms. When someone @mentions the bot in a channel, it spawns a Claude Code CLI session in a configured working directory and streams all output to a thread. The user can continue the conversation by replying in the thread.
---

# Claude Code Instructions for claude-threads

## What This Project Does

This is a multi-platform bot that lets users interact with Claude Code through chat platforms. When someone @mentions the bot in a channel, it spawns a Claude Code CLI session in a configured working directory and streams all output to a thread. The user can continue the conversation by replying in the thread.

**Currently Supported Platforms:**
- Mattermost (full support)
- Slack (full support)

**Key Features:**
- Real-time streaming of Claude responses to chat platforms
- **Multi-platform support** - connect to multiple Mattermost/Slack instances simultaneously
- **Multiple concurrent sessions** - one per thread, across all platforms
- **Session persistence** - sessions resume automatically after bot restart
- **Session collaboration** - `!invite @user` to temporarily allow users in a session
- **Message approval** - unauthorized users can request approval for their messages
- **Thread context prompt** - when starting a session mid-thread, offers to include previous conversation context
- Interactive permission approval via emoji reactions
- Plan approval and question answering via reactions
- Task list display with live updates
- Code diffs and file previews
- Multi-user access control
- Automatic idle session cleanup

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                      Chat Platform                               │
│  ┌──────────────┐                    ┌──────────────────────┐   │
│  │ User message │ ───WebSocket───▶  │   PlatformClient     │   │
│  │ + reactions  │ ◀───────────────  │   (Mattermost/Slack) │   │
│  └──────────────┘                    └──────────┬───────────┘   │
└─────────────────────────────────────────────────┼───────────────┘
                                                  │
                      ┌───────────────────────────┴────────────────┐
                      │              SessionManager                 │
                      │  - Orchestrates session lifecycle           │
                      │  - Delegates to specialized modules         │
                      │  - sessions: Map<sessionId, Session>        │
                      │  - postIndex: Map<postId, threadId>         │
                      └───────────────────────────┬────────────────┘
                                                  │
                              ┌───────────────────┼───────────────────┐
                              │                   │                   │
                              ▼                   ▼                   ▼
                      ┌───────────┐       ┌───────────┐       ┌───────────┐
                      │  Session  │       │  Session  │       │  Session  │
                      │ (thread1) │       │ (thread2) │       │ (thread3) │
                      └─────┬─────┘       └─────┬─────┘       └─────┬─────┘
                            │                   │                   │
                            ▼                   ▼                   ▼
                      ┌───────────┐       ┌───────────┐       ┌───────────┐
                      │ ClaudeCli │       │ ClaudeCli │       │ ClaudeCli │
                      │ + MCP srv │       │ + MCP srv │       │ + MCP srv │
                      └───────────┘       └───────────┘       └───────────┘
```

**Session contains:**
- `claude: ClaudeCli` - the Claude CLI process
- `claudeSessionId: string` - UUID for session persistence/resume
- `messageManager: MessageManager` - orchestrates all message operations and state
- `sessionAllowedUsers: Set<string>` - per-session allowlist (includes session owner)
- `isResumed: boolean` - whether session was resumed after restart

**MessageManager contains executors that own their state:**
- `ContentExecutor` - content streaming state
- `TaskListExecutor` - task list display state
- `QuestionApprovalExecutor` - pending questions/approvals
- `PromptExecutor` - context prompts, worktree prompts, update prompts
- `SubagentExecutor` - active subagent tracking
- `MessageApprovalExecutor` - unauthorized message approval
- `BugReportExecutor` - bug report flow

**MCP Permission Server:**
- Spawned via `--mcp-config` per Claude CLI instance
- Each has its own WebSocket/connection to the platform
- Posts permission requests to the session's thread
- Returns allow/deny based on user reaction

## Multi-Platform Support

**Architecture**: claude-threads supports connecting to multiple chat platforms simultaneously through a platform abstraction layer.

**Currently Supported**:
- ✅ Mattermost (fully implemented)
- ✅ Slack (fully implemented)

**Key Concepts**:

1. **Platform Abstraction**: `PlatformClient` interface normalizes differences between platforms
2. **Composite Session IDs**: Sessions are identified by `"platformId:threadId"` to ensure uniqueness across platforms
3. **Independent Credentials**: Each platform instance has its own URL, token, and channel configuration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anneschuth/claude-threads](https://github.com/anneschuth/claude-threads) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
