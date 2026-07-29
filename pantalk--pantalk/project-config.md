---
trigger: always_on
description: Pantalk can automatically launch AI agents when matching notifications arrive. Instead of polling for new messages, you define agents in your config and `pantalkd` triggers them reactively - buffering events, enforcing cooldowns, and restricting which binaries can run.
---

# Agents

Pantalk can automatically launch AI agents when matching notifications arrive. Instead of polling for new messages, you define agents in your config and `pantalkd` triggers them reactively - buffering events, enforcing cooldowns, and restricting which binaries can run.

Agents are **fire-and-forget**. When triggered, the command runs and reads notifications itself via `pantalk notifications`. No events are piped to stdin.

## Quick Example

```yaml
bots:
  - name: ops-bot
    type: slack
    bot_token: $SLACK_BOT_TOKEN
    app_level_token: $SLACK_APP_LEVEL_TOKEN

agents:
  - name: responder
    when: "direct || mentions"
    command: claude -p "Check pantalk notifications and respond"
    workdir: /home/user/project
```

When someone DMs the bot or @mentions it, `pantalkd` waits 30 seconds (batching), then launches `claude` with the given prompt. The agent calls `pantalk notifications --unseen` to read what happened and acts on it.

## Configuration

Each agent is defined under the `agents` key in your config:

```yaml
agents:
  - name: responder              # required - unique identifier
    when: "direct || mentions"   # expression (default: "notify")
    command: claude -p "Check notifications"  # required
    workdir: /home/user/project  # optional - inherits daemon's cwd
    buffer: 30                   # seconds to batch events (default: 30)
    timeout: 120                 # kill after N seconds (default: 120)
    cooldown: 60                 # min gap between runs (default: 60)
```

### Fields

| Field      | Required | Default    | Description                                               |
| ---------- | -------- | ---------- | --------------------------------------------------------- |
| `name`     | yes      | -          | Unique identifier, used in log messages                   |
| `when`     | no       | `"notify"` | Boolean expression evaluated against each event           |
| `command`  | yes      | -          | Binary + args to exec (string or array)                   |
| `workdir`  | no       | daemon cwd | Working directory for the command                         |
| `buffer`   | no       | `30`       | Seconds to wait and batch events before launching         |
| `timeout`  | no       | `120`      | Maximum runtime in seconds before the process is killed   |
| `cooldown` | no       | `60`       | Minimum seconds between consecutive runs of this agent    |

### Command Format

The `command` field accepts both a string and an array. It is **exec'd directly** - never passed through a shell.

```yaml
# String form - tokenized with shell-like quoting (no variable expansion)
command: claude -p "Check pantalk notifications and respond"

# Array form - each element is a separate argv entry
command:
  - claude
  - -p
  - "Check pantalk notifications and respond"
```

Both forms produce the same argv: `["claude", "-p", "Check pantalk notifications and respond"]`.

## When Expressions

The `when` field uses the [expr](https://github.com/expr-lang/expr) expression language. Expressions are boolean and evaluated against each inbound message event.

### Available Fields

**Event fields** - populated on message events, zero on tick events:

| Field      | Type   | Description                                      |
| ---------- | ------ | ------------------------------------------------ |
| `notify`   | bool   | Event is a notification (DM, mention, or thread) |
| `direct`   | bool   | Event is a direct message to the bot             |
| `mentions` | bool   | Event mentions the bot                           |
| `channel`  | string | Channel name or ID (e.g. `"#general"`)           |
| `thread`   | string | Thread ID (empty if not in a thread)             |
| `bot`      | string | Bot name from config                             |
| `service`  | string | Platform type (`"slack"`, `"discord"`, etc.)     |
| `user`     | string | User ID of the message author                    |
| `text`     | string | Message text content                             |

**Time fields** - populated on tick events (1-minute internal clock), zero on message events:

| Field      | Type   | Description                                      |
| ---------- | ------ | ------------------------------------------------ |
| `tick`     | bool   | True on clock tick events                        |
| `hour`     | int    | Current hour (0–23)                              |
| `minute`   | int    | Current minute (0–59)                            |
| `weekday`  | string | Day name: `"mon"`, `"tue"`, ..., `"sun"`         |

### Time Functions

| Function             | Description                                         |
| -------------------- | --------------------------------------------------- |
| `at("HH:MM", ...)`  | True when current time matches any listed time      |
| `every("Nm")`        | True on aligned minute intervals (e.g. :00, :15, :30, :45 for `"15m"`) |
| `every("Nh")`        | True on aligned hour intervals at minute :00        |

### Operators


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pantalk/pantalk](https://github.com/pantalk/pantalk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
