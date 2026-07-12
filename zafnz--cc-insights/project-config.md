---
trigger: always_on
description: CC-Insights is a desktop application for monitoring and interacting with Claude Code agents via the SDK. It provides real-time visibility into agent hierarchies, tool usage, and conversation flow.
---

# CC-Insights V2 - Project Guide

---

## Project Overview

CC-Insights is a desktop application for monitoring and interacting with Claude Code agents via the SDK. It provides real-time visibility into agent hierarchies, tool usage, and conversation flow.

**V2 Goals:**
- Git worktrees as a core concept (not bolt-on)
- Flexible panel-based UI
- Clean hierarchy: Project → Worktree → Chat → Conversation
- Preserve working components (Dart SDK, display widgets)

**Architecture:**
- **Agent SDK Core** (`agent_sdk_core/`): Shared types, interfaces, and transport abstraction used by all backends
- **Claude Dart SDK** (`claude_dart_sdk/`): Claude CLI backend - spawns Claude CLI as subprocess
- **Codex Dart SDK** (`codex_dart_sdk/`): OpenAI Codex backend - communicates via JSON-RPC
- **Frontend** (`frontend/`): Flutter desktop app (macOS) with Provider state management

**Communication:** Backend SDKs convert provider-specific wire formats into a unified `InsightsEvent` stream. The frontend consumes events through an `EventTransport` abstraction, sending commands back via `BackendCommand` types. See `docs/insights-protocol/` for the full protocol specification.

---

## Core Concepts

### Terminology

| Term | Definition |
|------|------------|
| **Project** | A git repository. Contains one primary worktree and zero or more linked worktrees. |
| **Worktree** | A git working tree with files. Has a path and a branch. |
| **Primary Worktree** | The worktree at the repository root (where `.git` lives). |
| **Linked Worktree** | A worktree created via `git worktree add`. Points back to the primary's `.git`. |
| **Chat** | A user-facing conversation unit. Belongs to a worktree. Contains conversations and optionally an active SDK session. |
| **Conversation** | A persistent log of messages/output. Survives session lifecycle. Each chat has a primary conversation and zero or more subagent conversations. |
| **Agent** | A runtime SDK entity. Exists only while a session is active. Links to a Conversation for output storage. |
| **Session** | Internal SDK concept. Users see "Chats", not "Sessions". |

### Hierarchy

```
Project: CC-Insights
├── Worktree (primary)
│   ├── Branch: main
│   └── Chats:
│       ├── Chat 1
│       │   ├── Primary Conversation
│       │   └── Subagent Conversations...
│       └── Chat 2
│
└── Worktree (linked)
    ├── Branch: feat-dark-mode
    └── Chats:
        └── Chat 1
```

### Key Rules

- A project has exactly one primary worktree (the repo root)
- Worktree branches are mutable (can change via `git checkout`)
- Conversations are persistent; Agents are runtime-only
- Users interact with the primary conversation; subagent conversations are read-only
- Subagents can still request permissions (routed to their conversation)

---

## Directory Structure

```
cc-insights/
├── agent_sdk_core/                   # Shared SDK types and interfaces
│   ├── lib/
│   │   ├── agent_sdk_core.dart       # Main export
│   │   └── src/
│   │       ├── backend_interface.dart # AgentSession/AgentBackend interfaces
│   │       ├── transport/            # Transport abstraction layer
│   │       │   ├── event_transport.dart  # EventTransport interface
│   │       │   └── in_process_transport.dart # In-process implementation
│   │       └── types/                # Shared type definitions
│   │           ├── backend_commands.dart  # BackendCommand sealed hierarchy
│   │           ├── backend_provider.dart  # BackendProvider enum (claude, codex, etc.)
│   │           ├── callbacks.dart
│   │           ├── content_blocks.dart
│   │           ├── control_messages.dart
│   │           ├── insights_events.dart   # InsightsEvent sealed hierarchy
│   │           ├── session_options.dart
│   │           ├── tool_kind.dart         # ToolKind enum (ACP-aligned)
│   │           └── usage.dart
│
├── claude_dart_sdk/                  # Claude CLI backend
│   ├── lib/
│   │   ├── claude_sdk.dart           # Main export (re-exports agent_sdk_core)
│   │   └── src/
│   │       ├── cli_process.dart      # CLI subprocess management
│   │       ├── cli_session.dart      # CLI session (emits InsightsEvents)
│   │       ├── cli_backend.dart      # CLI backend implementation
│   │       ├── backend_factory.dart  # Backend type selection
│   │       ├── sdk_logger.dart       # SDK logging
│   │       └── types/                # Re-export shims for agent_sdk_core types
│
├── codex_dart_sdk/                   # OpenAI Codex backend
│   ├── lib/
│   │   └── src/
│   │       ├── codex_process.dart    # Codex CLI subprocess management
│   │       ├── codex_session.dart    # Codex session (emits InsightsEvents)
│   │       ├── codex_backend.dart    # Codex backend implementation
│   │       └── json_rpc.dart         # JSON-RPC 2.0 protocol layer
│
├── frontend/                         # Flutter desktop app
│   ├── lib/
│   │   ├── main.dart
│   │   ├── config/
│   │   │   └── fonts.dart            # Font configuration
│   │   ├── models/                   # Data models
│   │   │   ├── project.dart
│   │   │   ├── worktree.dart
│   │   │   ├── chat.dart
│   │   │   ├── conversation.dart

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zafnz/cc-insights](https://github.com/zafnz/cc-insights) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
