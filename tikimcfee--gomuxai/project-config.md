---
trigger: always_on
description: You are an agent running in a tmux session. This file tells you how to coordinate with other agents.
---

# CLAUDE.md

You are an agent running in a tmux session. This file tells you how to coordinate with other agents.

---

## The Golden Rule

**The `agent` binary is your only interface to the outside world.**

Everything you need to do - spawn agents, send messages, read output, store state - goes through `agent`. If you find yourself wanting to do something the tool doesn't support, **STOP**. Don't hack around it. Report the limitation.

```bash
# YES - Use the tool
agent send helper "analyze this"
agent state set myname status "blocked"

# NO - Don't bypass the tool
echo "message" > /tmp/some-hack.txt  # BAD
tmux send-keys -t session "text"     # BAD - use agent keys
```

---

## Core Commands

### Spawning Agents

```bash
agent spawn <name> --backend claude [task-file]
```

**Flags:**
- `--backend <name>` - Backend to use (required). Built-in: `claude`, `aider`
- `--backend-config <file>` - Path to custom backend JSON config (alternative to `--backend`)
- `--task <name>` - Reference a task from the DB task repository (see Tasks section)
- `--skip-permissions` - Pass `--dangerously-skip-permissions` to Claude CLI

**Behavior:**
- Task file argument reads from filesystem; `--task` flag reads from DB
- The `AGENT_NAME` env var is automatically set in the spawned session
- You'll need to `send` the first message yourself (spawn doesn't auto-send)

```bash
# Spawn and give it work
agent spawn helper --backend claude
agent send helper "Read task at tasks/research.md and execute it"

# Spawn with a stored task from DB
agent spawn helper --backend claude --task research-auth

# Spawn with auto-approve (use with caution)
agent spawn auto-worker --backend claude --skip-permissions
```

### Sending Messages

**Two modes. Pick the right one.**

| Mode | Command | When |
|------|---------|------|
| Immediate | `agent send <name> "msg"` | Interrupt now, urgent |
| Queued | `agent queue <name> "msg"` | Wait until idle, polite |

```bash
# Urgent - sends immediately, may interrupt
agent send helper "STOP - there's a critical bug"

# Polite - waits for idle prompt
agent queue helper "When you're done, please review auth.go"

# Queue with explicit sender (for coordination/tracking)
agent queue helper --from coordinator "Priority task incoming"
```

**Queue flags:**
- `--from <sender>` - Sender ID. Defaults to `AGENT_NAME` env var, then "user"

**Wart:** `queue` requires `agent watch` running somewhere. If the watcher isn't running, messages sit forever.

### Reading Output

```bash
agent capture <name> --lines 100
```

Returns terminal output, ANSI codes stripped. Use `--raw` to keep them.

### Checking Status

```bash
agent status <name>
```

Returns: `idle`, `processing`, or `unknown`

**Wart:** Status detection uses regex patterns. It can be wrong. If status is `unknown`, the agent might be stuck at a prompt you don't recognize.

---

## Data Storage (All in SQLite)

Everything is in one database. No filesystem hunting.

### Tasks

Store reusable task definitions:

```bash
agent task set research-auth -f tasks/research-auth.md   # From file
agent task set quick-task "Do X, Y, Z"                   # Inline
agent task get research-auth                              # Retrieve
agent task list                                           # List all
agent task clear research-auth                            # Remove
```

Tasks exist independently of agents. Multiple agents can reference the same task.

### State (Key-Value)

Store your working state:

```bash
agent state set myname status "analyzing auth module"
agent state set myname progress "3/5 files done"
agent state get myname status
agent state list myname
agent state clear myname
```

**Use this for coordination.** Other agents can read your state:

```bash
# Helper writes its status
agent state set helper status "done"
agent state set helper result "Found 3 vulnerabilities"

# Coordinator reads it
agent state get helper status
agent state get helper result
```

**Behaviors:**
- State persists until explicitly cleared or the agent is cleaned up (cleanup deletes state and logs)
- Reading a non-existent key prints "No state found for X.Y" (not an error)

### Logs

Query captured output from DB (ANSI-stripped):

```bash
agent logs <name>                  # Last 50 entries
agent logs <name> --lines 100      # More entries
agent logs <name> --since 5m       # Since 5 minutes ago
```

### Capture vs Logs: When to Use Which

| Method | Source | Use When |
|--------|--------|----------|
| `capture` | Live terminal buffer | Need current screen state, checking prompts |
| `logs` | DB (persistent) | Need historical output, timestamped entries |

**Capture** is ephemeral - it's whatever's in the tmux scrollback right now. Good for status checks and reading recent output.

**Logs** are persistent - entries are stored with timestamps as the agent runs. Good for debugging what happened, auditing, or recovering context after the fact.

---

## Agent Identity

When an agent is spawned, the `AGENT_NAME` environment variable is set in its session. This is useful for:

1. **Self-identification** - Agent knows its own name without being told
2. **Queue sender defaults** - `agent queue` uses `AGENT_NAME` as default `--from`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tikimcfee/gomuxai](https://github.com/tikimcfee/gomuxai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
