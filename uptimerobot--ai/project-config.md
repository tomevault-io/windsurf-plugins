---
trigger: always_on
description: Orientation for coding agents (Claude Code, Cursor, Codex, Aider, etc.) that need to create, inspect, or control UptimeRobot monitors on a user's behalf.
---

# AGENTS.md — UptimeRobot

Orientation for coding agents (Claude Code, Cursor, Codex, Aider, etc.) that need to create, inspect, or control UptimeRobot monitors on a user's behalf.

## What UptimeRobot is

UptimeRobot monitors websites, APIs, servers, DNS, and services. It probes each target on an interval (minimum 30s on paid plans), records up/down state plus response times, opens incidents on downtime, and fires alerts (email, SMS, webhooks, Slack, etc.) through **alert contacts**.

Key entities:

- **Monitor** — one probe against one target. Has a `type` (HTTP, KEYWORD, PING, PORT, HEARTBEAT, DNS, API, UDP), a URL/host, an interval, and a status (`UP`, `DOWN`, `PAUSED`, `NOT_STARTED`, etc.).
- **Incident** — a downtime event. Opens when a monitor goes down, closes when it recovers. Has logs, checker IPs, traceroute.
- **Alert contact** — a destination (email, Slack, webhook, SMS, Telegram, etc.) attached to monitors with a threshold and recurrence.
- **Tag** — free-form label assigned to monitors.
- **Maintenance window** — scheduled period where alerts for attached monitors are suppressed.

Plans: **Free, Solo, Team, Enterprise**. Availability of specific monitor types and minimum intervals depends on the active plan. The MCP server enforces all limits.

## How to talk to UptimeRobot

Use the remote MCP server:

- URL: `https://mcp.uptimerobot.com/mcp`
- Transport: HTTP (streamable), launched via `mcp-remote` (`npx -y mcp-remote@latest https://mcp.uptimerobot.com/mcp`).
- Auth: **OAuth**. On first connection `mcp-remote` opens a browser to log into UptimeRobot and authorize — no API key to paste. (For headless/CI where a browser can't run, connect over plain HTTP with an `Authorization: Bearer <API_KEY>` header instead.)

All 32 tools operate on the authorized account. There is no account impersonation.

First-time setup (Claude Code, Cursor, or any MCP client): see [`skills/setup/SKILL.md`](skills/setup/SKILL.md).

## No account? Create a monitor with just an email

If the user has no UptimeRobot account or API key, use the [`quick-monitor-setup`](skills/quick-monitor-setup/SKILL.md) skill instead of the MCP tools: it creates a free HTTPS monitor from just the owner's email (an unauthenticated proof-of-work flow; the owner confirms by clicking a link in an activation email). When the user is connected via MCP, use `create-monitor`.

## Tools (all 32)

Grouped by resource. Tools marked **plan-gated** require a specific plan feature (`monitor-groups`, `maintenance-window`, `incident-comments`, or `psp-subscribers`) and fail if the account's plan doesn't include it.

### Monitors

| Name | When to use |
| -- | -- |
| `create-monitor` | Create a new monitor of any supported type. See [`skills/create-*-monitor`](skills/) for per-type recipes. |
| `update-monitor` | Partially update an existing monitor's name, URL, interval, alert contacts, tags, HTTP settings, etc. Monitor type cannot be changed. See [`skills/update-monitor/SKILL.md`](skills/update-monitor/SKILL.md). |
| `update-monitor-status` | Pause (`PAUSED`) or resume (`STARTED`) a monitor. |
| `list-monitors` | Browse monitors with optional search string and state filters (`UP`, `DOWN`, `PAUSED`, `NOT_STARTED`, `EXPIRING_DOMAIN`, `EXPIRING_SSL_CERTIFICATE`, `WITH_API_KEY`, `WITHOUT_API_KEY`). Cursor-paginated. |
| `get-monitor-details` | Full config + current state for one monitor ID. |
| `get-monitor-stats` | Aggregated up/down/paused counts + overall uptime % for a time range. |
| `get-response-times` | Time-series response-time data for one monitor within a time range. Supports `bucketSize` for aggregation. |

### Monitor groups (plan-gated)

| Name | When to use |
| -- | -- |
| `create-monitor-group` | Create a group. Use `monitorIds` to assign monitors, or `groupIds` to move monitors out of existing groups into the new one. Group membership can only be set here or at monitor-creation time — see [`skills/manage-monitor-groups/SKILL.md`](skills/manage-monitor-groups/SKILL.md). |
| `update-monitor-group` | Rename an existing group. |
| `get-monitor-group` | Fetch a single group by `monitorGroupId`. |
| `list-monitor-groups` | Cursor-paginated list of groups. |

### Status pages (PSPs)

| Name | When to use |
| -- | -- |
| `create-psp` | Create a public status page from `monitorIds`, `monitorGroupIds`, or `tagIds`. Logo/icon uploads aren't supported over MCP — use the dashboard. See [`skills/status-pages/SKILL.md`](skills/status-pages/SKILL.md). |
| `update-psp` | Partially update name, custom domain, sort order, visibility (`noIndex`, `password`), or publish status (`status: ENABLED`/`PAUSED`). |
| `get-psp` | Fetch a single status page by `pspId`. |
| `list-psps` | Cursor-paginated list of status pages. |

### PSP announcements (plan-gated: `psp-subscribers`)

| Name | When to use |
| -- | -- |
| `create-psp-announcement` | Post an announcement (`Info`, `Maintenance`, or `Issue`) to a status page. Set `status: Published` to make it visible immediately. |
| `update-psp-announcement` | Partially update title, content, dates, type, or status (`Offline`/`Pending`/`Published`/`Archived`). |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uptimerobot/ai](https://github.com/uptimerobot/ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
