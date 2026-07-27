---
trigger: always_on
description: Pilot's **AI agent** is a context-aware coding assistant powered by the Pi SDK. It can read and write files, execute commands, manage tasks, and maintain memory across sessions — all within a sandboxed environment that requires your approval for file changes.
---

# Agent

Pilot's **AI agent** is a context-aware coding assistant powered by the Pi SDK. It can read and write files, execute commands, manage tasks, and maintain memory across sessions — all within a sandboxed environment that requires your approval for file changes.

---

## What is the Agent?

The agent is:
- **AI-powered**: Uses large language models (OpenAI, Anthropic, Google, etc.)
- **Tool-equipped**: Can read/write files, run bash commands, query git, manage tasks
- **Context-aware**: Has access to project memory, file structure, git history, task lists
- **Sandboxed**: All file modifications are staged for your review before being applied
- **Conversational**: Responds to natural language requests and questions

---

## How the Agent Works

### 1. You Send a Message

Type your request in the chat input:
```
Refactor the IPC handlers to use async/await
```

### 2. Agent Analyzes Context

The agent receives:
- **Your message**
- **Chat history** (previous conversation in this session)
- **Memory context** ([global and project](./memory.md))
- **System prompt** (instructions on how to assist you)
- **Tool definitions** (available tools and their parameters)

### 3. Agent Decides on Actions

The agent chooses to:
- **Respond directly** (answer a question, provide guidance)
- **Use tools** (read files, execute commands, create tasks)
- **Ask for clarification** (if the request is ambiguous)

### 4. Tool Execution

If the agent uses tools:
- **Read-only tools** (`read`, `bash`, `pilot_task_query`) execute immediately
- **Write tools** (`write`, `edit`, `pilot_task_create`) are **sandboxed** — staged for review

### 5. You Review Changes

For sandboxed operations:
- **Diff panel** opens in the [Context Panel](./context-panel.md#changes-tab)
- You see the proposed changes (before/after)
- You **accept** or **reject** the changes
- Agent continues after your decision

### 6. Agent Responds

The agent:
- Explains what it did
- Shows the results of tool executions
- Asks follow-up questions if needed
- Waits for your next message

---

## Steering & Follow-up

While the agent is actively working, you can redirect it or queue follow-up instructions without waiting:

- **Steer** (`Enter` while streaming) — Interrupts after the current tool. The agent sees your message immediately.
- **Follow-up** (`Alt+Enter` while streaming) — Queued until the agent finishes all current work.
- **Stop** (click stop button) — Aborts entirely.

Pending messages appear as colored pills above the input. See the full **[Steering & Follow-up guide](steering.md)** for details, examples, and tips.

---

## Agent Tools

### File Tools

#### `read`

**Purpose**: Read file contents.

**Usage**:
```
Read src/app.tsx
```

**Behavior**:
- Returns full file contents
- Truncated if very large (>50KB)
- Not sandboxed (no approval needed)

#### `write`

**Purpose**: Create a new file or overwrite an existing file.

**Usage**:
```
Create a new component at src/components/TaskCard.tsx
```

**Behavior**:
- **Sandboxed** — staged for review
- Diff shows full file contents (create = all green lines)
- Applied to disk only after you accept

#### `edit`

**Purpose**: Make surgical edits to an existing file (find and replace).

**Usage**:
```
In src/app.tsx, replace the old event handler with the new one
```

**Behavior**:
- **Sandboxed** — staged for review
- Diff shows only the changed lines (context included)
- More precise than `write` for small changes
- Fails if the "old text" doesn't match exactly (prevents accidental edits)

### Shell Tools

#### `bash`

**Purpose**: Execute shell commands in the project directory.

**Usage**:
```
Run npm install to add the new dependency
```

**Behavior**:
- Executes in the project root
- Returns stdout and stderr
- **Sandboxed for destructive commands** (rm, mv, etc.) — you must approve
- Read-only commands (ls, cat, grep) execute immediately

**Security**: Commands are restricted to the project directory (cannot escape the project jail).

**Bash Jail Enforcement**:

When the project jail is enabled, every bash command is analyzed before execution to prevent path escapes:

1. **Path Extraction** — The system scans the command for:
   - Absolute paths (`/home/user/file.txt`)
   - Home directory references (`~/file.txt`, `$HOME/file.txt`)
   - Parent directory escapes (`../outside-project/file.txt`)
   - Redirect targets (`> /tmp/output.txt`)
   - Paths in quotes or command substitution
   - Environment variables (`$TMPDIR`, `$HOME`)

2. **Path Validation** — Each extracted path is checked:
   - Must be within the project root OR in the allowed paths list (`.pilot/settings.json`)
   - System paths are implicitly allowed (`/usr`, `/bin`, `/dev`, `/etc`, `/opt`, etc.)
   - Environment variables are expanded before checking

3. **Enforcement** — If any path escapes:
   - The command is **blocked immediately**
   - A clear error message lists the offending paths
   - The agent is told to add the paths to `allowedPaths` in `.pilot/settings.json`

4. **Normal Flow** — If all paths are clean:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [espennilsen/pilot](https://github.com/espennilsen/pilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
