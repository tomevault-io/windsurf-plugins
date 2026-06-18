---
trigger: always_on
description: Calendly scheduling integration. List events, check availability, manage meetings via Calendly API.
---


# Calendly Skill

Interact with Calendly scheduling via MCP-generated CLI.

> **Note:** This CLI includes `schedule-event` and `reschedule-event` with strict input validation and MCP-first execution. If MCP scheduling is unavailable, each command falls back to Calendly REST.

## Quick Start

```bash
# Make the launcher available on PATH
bun run install:path
export PATH="$HOME/.local/bin:$PATH"
command -v calendly

# Get your Calendly profile (returns user URI)
calendly get-current-user

# List RECENT events (always use --min-start-time for recent queries!)
calendly list-events \
  --user-uri "<YOUR_USER_URI>" \
  --min-start-time "2026-01-20T00:00:00Z" \
  --max-start-time "2026-01-27T23:59:59Z"

# Get event details
calendly get-event --event-uuid <UUID>

# Cancel an event
calendly cancel-event --event-uuid <UUID> --reason "Rescheduling needed"
```

Default install target: `~/.local/bin/calendly`.
Override it with `bun run install:path -- --bin-dir /your/bin/dir`.

## Available Commands

### User Info
- `get-current-user` - Get authenticated user details

### Events
- `list-events` - List scheduled events (requires --user-uri); add `--include-invitees` for expand + bounded fallback invitee hydration
- `list-events-with-invitees` - Compatibility alias for include-invitees path
- `get-event` - Get event details (requires --event-uuid)
- `list-event-types` - List schedulable event types (requires `--user-uri` or `--organization-uri`; optional `--count`)
- `get-event-type` - Get event type details (requires `--event-type-uri`)
- `update-event-type` - Update mutable event type metadata (`--event-type-uri` primary, `--event-type-uuid` alias, supports `--dry-run`)
- `get-event-type-availability` - Get available slots for an event type (`--event-type-uri`, `--start-time`, `--end-time`; optional `--timezone`)
- `schedule-event` - Schedule a meeting by booking an invitee into an event type
- `reschedule-event` - Reschedule an existing meeting to a new start time using event/invitee identifiers
- `cancel-event` - Cancel an event (requires --event-uuid, optional --reason)

### Invitees
- `list-event-invitees` - List invitees for an event (requires --event-uuid)
- `batch-event-invitees` - Batch lookup invitees for multiple events (repeat `--event-uri`)
- `search-invitees` - Search events by invitee email across paginated results

### Team Events
- `list-team-events` - List scheduled events for a team by scanning organization memberships and member calendars

### Organization
- `list-organization-memberships` - List organization memberships

### Webhooks
- `list-webhook-subscriptions` - List webhook subscriptions
- `get-webhook-subscription` - Get webhook subscription details
- `create-webhook-subscription` - Create webhook subscription
- `delete-webhook-subscription` - Delete webhook subscription

## Configuration

API key can be stored in your environment or `.env` file:
```bash
export CALENDLY_API_KEY="<your-pat-token>"
# Or in legacy ~/.moltbot/.env / ~/.clawdbot/.env
```

Get your Personal Access Token from: https://calendly.com/integrations/api_webhooks

## Usage in OpenClaw

When user asks about:
- "What meetings do I have?" → `list-events` with `--min-start-time` (use recent date!)
- "Show me all demos this week with who booked them" → `list-events --include-invitees` (expand first; fallback hydrate only when needed)
- "Cancel my 2pm meeting" → Find with `list-events` (time-filtered), then `cancel-event`
- "Who's attending X meeting?" → `list-events --include-invitees` (with fallback) or `list-event-invitees`

**Note:** First time, run `calendly get-current-user` to obtain your User URI.

## Getting Your User URI

Run `calendly get-current-user` to get your user URI. Example:
```json
{
  "resource": {
    "uri": "https://api.calendly.com/users/<YOUR_USER_UUID>",
    "scheduling_url": "https://calendly.com/<your-username>"
  }
}
```

## Examples

```bash
# List next 10 events
calendly list-events \
  --user-uri "<YOUR_USER_URI>" \
  -o json | jq .

# List events with invitees (expand first; bounded fallback)
calendly list-events \
  --user-uri "<YOUR_USER_URI>" \
  --include-invitees \
  --max-invitee-fetches 25 \
  --status active

# List team events for a shared calendar/org scope
calendly list-team-events \
  --organization-uri "<YOUR_ORG_URI>" \
  --min-start-time "2026-01-20T00:00:00Z" \
  --max-start-time "2026-01-27T23:59:59Z" \
  --include-invitees

# Get event details
calendly get-event \
  --event-uuid "<EVENT_UUID>" \
  -o json

# Get event type details
calendly get-event-type \
  --event-type-uri "https://api.calendly.com/event_types/<EVENT_TYPE_UUID>" \
  -o json

# Update event type metadata
calendly update-event-type \
  --event-type-uri "https://api.calendly.com/event_types/<EVENT_TYPE_UUID>" \
  --duration 30 \
  --active true \
  -o json

# Dry-run without applying
calendly update-event-type \
  --event-type-uuid "<EVENT_TYPE_UUID>" \
  --description "Updated invitee-facing description" \
  --dry-run \
  -o json

# Get event type availability
calendly get-event-type-availability \
  --event-type-uri "https://api.calendly.com/event_types/<EVENT_TYPE_UUID>" \
  --start-time "2026-03-01T00:00:00Z" \
  --end-time "2026-03-02T00:00:00Z" \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kesslerio/calendly-cli-openclaw-skill](https://github.com/kesslerio/calendly-cli-openclaw-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
