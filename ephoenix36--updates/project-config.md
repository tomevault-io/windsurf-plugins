---
trigger: always_on
description: **Skill name:** `updates-cli`
---

# Updates CLI Skill

**Skill name:** `updates-cli`  
**Version:** 2.0.0  
**Description:** Full-featured CLI for the Updates notification, scheduling, and timer system.

> **When to invoke this skill:**  
> Use when the user or an automated workflow needs to:  
> - Send a notification to any configured channel (ntfy, Discord, Slack, Telegram, email, SMS, webhook, etc.)  
> - Manage notification channels (create, list, test, enable/disable, delete)  
> - Manage schedules (one-time, interval, or cron-based)  
> - Manage timers (create, start, stop, pause, resume, reset)  
> - Browse notification history and retry failed deliveries  
> - Check system health and statistics

---

## Prerequisites

The Updates backend must be running and reachable (default: `http://127.0.0.1:8000`).

```bash
# Start the backend (Windows PowerShell — from the project root)
.\launch.ps1

# Or start directly
cd backend && uv run uvicorn updates.api.main:app --host 127.0.0.1 --port 8000
```

Install the CLI once:

```bash
cd backend
uv pip install -e .
updates-cli --help
```

Set the API URL if the backend is not on localhost:

```bash
export UPDATES_API_URL=http://myserver:8000   # Linux/macOS/WSL
$env:UPDATES_API_URL = "http://myserver:8000" # Windows PowerShell
```

---

## Complete Command Reference

### Notifications

| Command | Description |
|---|---|
| `send <channel> <message> [--title T] [--priority P]` | Send a notification |
| `history [--limit N] [--status sent\|failed\|pending]` | List recent notifications |
| `notification-retry <id>` | Retry a failed notification |
| `notification-delete <id>` | Delete a notification record |

### Channels

| Command | Description |
|---|---|
| `channels` | List all channels |
| `channel-create --name X --type Y [--set K=V ...] [--disabled]` | Create a channel |
| `channel-test <name-or-id>` | Send a test message via a channel |
| `channel-enable <name-or-id>` | Enable a disabled channel |
| `channel-disable <name-or-id>` | Disable a channel |
| `channel-delete <name-or-id>` | Delete a channel |

**Channel types and their `--set` keys:**

| type | required `--set` keys |
|---|---|
| `ntfy` | `ntfy_topic` (and optionally `ntfy_server`, `ntfy_token`) |
| `discord` | `discord_webhook_url` |
| `slack` | `slack_webhook_url` |
| `telegram` | `telegram_bot_token`, `telegram_chat_id` |
| `email` | `smtp_host`, `smtp_port`, `smtp_username`, `smtp_password`, `from_email`, `to_email` |
| `sms` | `twilio_account_sid`, `twilio_auth_token`, `twilio_from_number`, `twilio_to_number` |
| `webhook` | `webhook_url` (optionally `webhook_method`) |
| `phone_call` | `call_from_number`, `call_to_number` |

### Schedules

| Command | Description |
|---|---|
| `schedules` | List all schedules |
| `schedule-create ...` (see below) | Create a schedule |
| `schedule-enable <id>` | Enable a schedule |
| `schedule-disable <id>` | Disable a schedule |
| `schedule-trigger <id>` | Run a schedule immediately |
| `schedule-delete <id>` | Delete a schedule |

**`schedule-create` flags:**
```
--name X          schedule name (required)
--channel X       channel name or id (required)
--type once|interval|cron  (required)
--title T         notification title (required)
--message M       notification message (required)
--at ISO8601      datetime for once-type (e.g. 2026-04-06T09:00:00)
--interval-seconds N  interval in seconds, min 60
--cron EXPR       cron expression (e.g. "0 9 * * *")
--disabled        create in disabled state
```

### Timers

| Command | Description |
|---|---|
| `timers` | List all timers |
| `timer-create --name X --duration N [--channel C] [--start]` | Create (and optionally start) a timer |
| `timer-start <id>` | Start a stopped/paused timer |
| `timer-stop <id>` | Stop a timer |
| `timer-pause <id>` | Pause a running timer |
| `timer-resume <id>` | Resume a paused timer |
| `timer-reset <id>` | Reset a timer to its full duration |
| `timer-delete <id>` | Delete a timer |

### System

| Command | Description |
|---|---|
| `status` | Show backend health, channel/notification/timer stats |

---

## Tool Definitions (Anthropic tool_use format)

### `updates_send`

```json
{
  "name": "updates_send",
  "description": "Send a notification to a configured channel (ntfy, Discord, Slack, Telegram, email, SMS, webhook, etc.).",
  "input_schema": {
    "type": "object",
    "properties": {
      "channel": {"type": "string", "description": "Channel name or numeric id"},
      "message": {"type": "string"},
      "title":   {"type": "string", "description": "Optional title (default: 'Notification')"},
      "priority":{"type": "string", "enum": ["low","normal","high","urgent"], "default": "normal"}
    },
    "required": ["channel", "message"]
  }
}
```

### `updates_channels`

```json
{
  "name": "updates_channels",
  "description": "List all configured notification channels. Use before updates_send to discover channel names.",
  "input_schema": {"type": "object", "properties": {}, "required": []}
}
```

### `updates_channel_create`

```json
{
  "name": "updates_channel_create",
  "description": "Create a new notification channel.",
  "input_schema": {
    "type": "object",
    "properties": {
      "name":     {"type": "string"},

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ephoenix36/Updates](https://github.com/ephoenix36/Updates) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
