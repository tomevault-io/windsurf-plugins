---
trigger: always_on
description: This document compares the AI coding agents supported by `agent-history` and explains how they work with this tool.
---

# Supported Coding Agents

This document compares the AI coding agents supported by `agent-history` and explains how they work with this tool.

## Quick Comparison

| Feature | Claude Code | Codex CLI | Gemini CLI | Pi |
|---------|-------------|-----------|------------|----|
| **Developer** | Anthropic | OpenAI | Google | Pi |
| **Session Format** | JSONL | JSONL | JSON | JSONL |
| **Storage Location** | `~/.claude/projects/` | `~/.codex/sessions/` | `~/.gemini/tmp/` | `~/.pi/agent/sessions/` |
| **Organization** | By workspace path | By date (YYYY/MM/DD) | By project hash | By workspace path |
| **Workspace ID** | Encoded path | Extracted from session | SHA-256 of path | Session `cwd` or encoded path |
| **Built-in Export** | None | None | `/chat share` | `pi agent session export` |
| **Token Tracking** | Per-message | Per-turn | Per-message | Per-message when present |
| **Reasoning/Thoughts** | Not stored | Not stored | Stored | Stored when present |

## Storage Locations

### Claude Code

```
~/.claude/projects/
└── -home-user-myproject/           # Encoded workspace path
    ├── <uuid>.jsonl                # Main conversation
    └── agent-<id>.jsonl            # Task subagent sessions
```

- **Workspace naming**: Path encoded with dashes (e.g., `/home/user/myproject` → `-home-user-myproject`)
- **Session files**: UUID-named JSONL files
- **Subagents**: Separate files prefixed with `agent-`

### Codex CLI

```
~/.codex/sessions/
└── 2025/12/15/                     # Date-based organization
    └── rollout-<timestamp>.jsonl   # Session file
```

- **Workspace naming**: Extracted from `cwd` field in session metadata
- **Session files**: Timestamp-prefixed JSONL files
- **Date organization**: Sessions grouped by YYYY/MM/DD folders

### Gemini CLI

```
~/.gemini/tmp/
└── <sha256-hash>/                  # Hash of project path
    └── chats/
        └── session-<id>.json       # Session file (single JSON)
```

- **Workspace naming**: SHA-256 hash of absolute project path
- **Session files**: JSON files (not JSONL) containing full session
- **Hash index**: `agent-history` maintains a hash→path index for readable display

### Pi

```
~/.pi/agent/sessions/
└── --home-user-myproject--/        # Encoded workspace path
    └── <timestamp>_<id>.jsonl      # Session file
```

- **Workspace naming**: Read from the session `cwd` header when available; otherwise decoded from the workspace folder
- **Session files**: JSONL files with a `session` header and `message` entries
- **Tool calls**: Assistant tool calls and tool/batch execution results are preserved

## How agent-history Works with Each Agent

### Listing Sessions (`lss`)

```bash
# All agents (auto-detect)
agent-history lss myproject

# Specific agent
agent-history --agent claude lss myproject
agent-history --agent codex lss myproject
agent-history --agent gemini lss myproject
agent-history --agent pi lss myproject
```

| Behavior | Claude | Codex | Gemini | Pi |
|----------|--------|-------|--------|----|
| Pattern matching | On encoded path | On workspace path | On path or hash | On workspace path |
| Date filtering | File mtime | File mtime | File mtime | File mtime |
| Message count | From JSONL | From JSONL | From JSON | From JSONL |

### Exporting Sessions (`export`)

```bash
# Export to markdown
agent-history export myproject -o ./output

# Agent-specific export
agent-history --agent gemini export myproject
```

| Feature | Claude | Codex | Gemini | Pi |
|---------|--------|-------|--------|----|
| Output format | Markdown/HTML | Markdown/HTML | Markdown/HTML | Markdown/HTML |
| Metadata | Full (UUIDs, tokens, etc.) | Basic (workspace, timestamps) | Full (tokens, thoughts) | Session header and message metadata |
| Tool calls | Preserved | Preserved | Preserved | Preserved |
| Reasoning steps | N/A | N/A | Included | Included when present |

### Statistics (`stats`)

```bash
# Sync and show stats
agent-history stats --sync
agent-history stats --tools
agent-history stats --models
```

| Metric | Claude | Codex | Gemini | Pi |
|--------|--------|-------|--------|----|
| Token counts | Input/output/cache | Input/output | Input/output/thoughts | Input/output when present |
| Tool usage | Full tracking | Full tracking | Full tracking | Full tracking |
| Model info | Yes | Yes | Yes | Yes when present |
| Work time | Calculated | Calculated | Calculated | Calculated |

## Agent-Specific Features

### Claude Code

- **Subagent tracking**: Task tool spawns separate agent sessions linked by parent ID
- **Cache tokens**: Tracks cache creation and read tokens
- **Git integration**: Records git branch in session metadata
- **Version tracking**: Stores Claude Code version

### Codex CLI

- **Incremental indexing**: `agent-history` maintains session→workspace index for O(1) lookups
- **Date-based scanning**: Only scans new date folders since last run
- **CLI version**: Stores Codex CLI version in sessions

### Gemini CLI

- **Reasoning/thoughts**: Captures model's reasoning steps with subjects and descriptions
- **Hash→path index**: `agent-history` progressively learns hash→path mappings
- **Built-in export**: Gemini has `/chat share` command (we provide more features)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kvsankar/agent-history](https://github.com/kvsankar/agent-history) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
