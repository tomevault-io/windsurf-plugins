---
trigger: always_on
description: Agents are BunkerAI-powered automations that run on your broker. They let you set up reactive monitoring (Watchers) and time-based automation (Scheduled Jobs) without writing any code. Agents are configured and managed from the BunkerM dashboard.
---

# Tools - Agents

Agents are BunkerAI-powered automations that run on your broker. They let you set up reactive monitoring (Watchers) and time-based automation (Scheduled Jobs) without writing any code. Agents are configured and managed from the BunkerM dashboard.

!!! info "BunkerAI required"
    Agents require BunkerAI Cloud to be connected. Go to **Settings > Integrations** to connect your BunkerAI account. Community plan users can create up to 2 agents total.

---

## Two Types of Agents

| Type | What it does |
|------|-------------|
| **Watcher** | Monitors a topic and fires an action when a condition is met |
| **Scheduled Job** | Publishes a message to a topic on a cron schedule |

---

## Watchers

### What a Watcher Does

A Watcher subscribes to an MQTT topic on your broker and continuously evaluates incoming messages against a condition you define. When the condition is met, the Watcher fires a configured response - logging an event, sending an alert, or triggering a follow-up action.

Watchers are ideal for:
- Detecting when sensor readings cross a threshold
- Alerting when a device publishes an unexpected value
- Monitoring heartbeat topics for missed messages
- Detecting state changes on actuators or switches

### Creating a Watcher

1. Navigate to **Tools > Agents** in the sidebar.
2. Click **Add Watcher**.
3. Configure the following fields:

**Topic** - The MQTT topic to watch. Can be a specific topic (`sensor/temp/kitchen`) or a pattern (`sensor/+/temp`).

**Condition Operator** - How to compare the incoming message value:

| Operator | Description |
|----------|-------------|
| `>` | Fires when value is greater than threshold |
| `<` | Fires when value is less than threshold |
| `=` | Fires when value exactly equals threshold |
| `!=` | Fires when value does not equal threshold |
| `>=` | Fires when value is greater than or equal to threshold |
| `<=` | Fires when value is less than or equal to threshold |
| `contains` | Fires when payload contains the specified string |

**Condition Value** - The threshold or value to compare against (e.g., `35`, `"error"`, `"offline"`).

**Response Template** - A message or action description that is logged when the watcher fires. You can reference the triggering message in the template.

**One-shot vs Continuous** - Choose whether the watcher fires once and then stops (one-shot) or continues firing every time the condition is met (continuous).

**Cooldown** - For continuous watchers, set a minimum time between firings in seconds to prevent alert flooding. For example, a 300-second cooldown means the watcher can fire at most once every 5 minutes even if the condition remains true.

4. Click **Save**.

The Watcher starts immediately and will fire the next time a matching message arrives that satisfies the condition.

### One-shot vs Continuous Watchers

**One-shot:** The watcher fires once when the condition is first met, then automatically disables itself. Use this for "fire and forget" scenarios, like sending a one-time alert when a device reports a critical value.

**Continuous:** The watcher keeps watching and fires every time a new message satisfies the condition (subject to cooldown). Use this for ongoing monitoring like high temperature alerts.

### Cooldown Settings

The cooldown prevents alert flooding when a condition stays true for an extended period. If `sensor/temp/kitchen` is above 35C and the device publishes every 10 seconds, a watcher with no cooldown would fire every 10 seconds. Set a 300-second cooldown to cap it at once every 5 minutes.

### Watcher Events Log

Every time a Watcher fires, an event is logged with:
- Timestamp
- Topic that triggered the event
- The payload value that matched the condition
- Which watcher fired

View the event log from the Agents page. Events also trigger the real-time bell notification in the BunkerM header.

### Example: Temperature Watcher

**Goal:** Get an alert whenever the kitchen temperature sensor reports above 35C.

- Topic: `sensor/temp/kitchen`
- Condition: `>` `35`
- Response: `Kitchen temperature exceeded 35C - current value: {value}`
- Mode: Continuous
- Cooldown: 600 seconds

---

## Scheduled Jobs

### What a Scheduled Job Does

A Scheduled Job publishes an MQTT message to a specified topic on a cron schedule. It runs automatically in the background without any external trigger. Use Scheduled Jobs for:

- Heartbeat messages to verify your broker is alive
- Periodic configuration updates to devices
- Timed commands (turn off lights at midnight, run a daily report)
- Test message injection during development

### Creating a Scheduled Job

1. Navigate to **Tools > Agents** in the sidebar.
2. Click **Add Scheduled Job**.
3. Configure the following fields:

**Description** - A human-readable name for this job (e.g., "Daily heartbeat", "Midnight light off").

**Cron Expression** - When the job runs. Standard 5-field cron format: `minute hour day-of-month month day-of-week`.

**Topic** - The MQTT topic to publish to.

**Payload** - The message content to publish.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bunkeriot/BunkerM](https://github.com/bunkeriot/BunkerM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
