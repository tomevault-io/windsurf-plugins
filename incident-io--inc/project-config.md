---
trigger: always_on
description: `inc` is a CLI for the [incident.io](https://incident.io) API. It manages incidents, alerts, catalog entries, escalations, schedules, and more. Every API command supports `--output json` for machine-readable output (`config` and `auth login|token` print plain text).
---

# AGENTS.md — Using `inc` (incident.io CLI)

`inc` is a CLI for the [incident.io](https://incident.io) API. It manages incidents, alerts, catalog entries, escalations, schedules, and more. Every API command supports `--output json` for machine-readable output (`config` and `auth login|token` print plain text).

## Authentication

Set the `INCIDENT_API_KEY` environment variable. That's it.

```bash
export INCIDENT_API_KEY=inc_abc123...
```

Verify it works:

```bash
inc auth status
```

## Command Grammar

```
inc <resource> <action> [flags]
```

Resources are plural nouns. Actions are usually verbs (exceptions: `schedules entries`, `post-mortems content`, `auth status|token`). Examples:

```bash
inc incidents list --status-category live
inc incidents show INC-123
inc catalog entries list --type-id 01HXYZ
inc escalations create --title "Database latency spike" --escalation-path-id 01HXYZ
```

## Output

Always use `--output json` when calling `inc`. Never parse table output.

```bash
# Full JSON
inc incidents list --output json

# Filter with --jq to reduce token usage (typed commands return unwrapped arrays)
inc incidents list --output json --jq '.[] | {id, name}'

# Limit fields to save tokens
inc incidents list --output json --fields id,name,reference,incident_status,severity

# Note: `inc api` returns raw enveloped responses, so use .incidents[] there:
inc api GET /v2/incidents --output json --jq '.incidents[] | {id, name}'
```

## The `inc api` Escape Hatch

`inc api` can hit any incident.io API endpoint directly. Use it when a dedicated command doesn't exist yet, or when you need fine-grained control.

```bash
# GET
inc api GET /v2/incidents --jq '.incidents[] | {id, name}'

# GET with query params
inc api GET /v2/incidents --field status[one_of]=live --field page_size=10

# POST with flags
inc api POST /v2/incidents --field name="Database outage" --field severity_id="01HXYZ"

# POST with stdin
echo '{"name": "Database outage"}' | inc api POST /v2/incidents --input -

# Paginate through all results
inc api GET /v2/incidents --paginate --jq '.incidents[]'
```

## Key Commands

| Command | Description |
|---------|-------------|
| `inc incidents list` | List incidents. Filter with `--status-category`, `--severity-id`, `--sort-by`. |
| `inc incidents show ID` | Get a single incident. `ID` is a ULID or a reference (`INC-123`). |
| `inc incidents create` | Create an incident. |
| `inc incidents update ID` | Update an incident. Use `--notify=false` for silent updates. `ID` accepts a reference. |
| `inc incidents close ID` | Close an incident. `ID` accepts a reference. |
| `inc alerts list` | List alerts. |
| `inc alerts show ID` | Get a single alert. |
| `inc catalog types list` | List catalog types. |
| `inc catalog types show ID` | Get a single catalog type. |
| `inc catalog types create` | Create a catalog type. Requires `--name`, `--description`. |
| `inc catalog types delete ID` | Delete a catalog type. |
| `inc catalog entries list --type-id TYPE` | List catalog entries of a given type. |
| `inc catalog entries show ID` | Get a single catalog entry. |
| `inc catalog entries create` | Create a catalog entry. Requires `--type-id`, `--name`. |
| `inc catalog entries update ID` | Update a catalog entry. Requires `--name`. |
| `inc catalog entries delete ID` | Delete a catalog entry. |
| `inc escalations list` | List live escalations. |
| `inc escalations show ID` | Get a single escalation. |
| `inc escalations create` | Trigger an escalation. Requires `--title`. |
| `inc escalations paths list` | List escalation path configurations. |
| `inc escalations paths show ID` | Get a single escalation path. |
| `inc schedules list` | List on-call schedules. |
| `inc schedules show ID` | Get a single schedule. |
| `inc schedules entries ID` | List who is on call. Use `--from` and `--until` for time window. |
| `inc schedules override` | Create a schedule override (swap a shift). |
| `inc severities list` | List severities. |
| `inc severities show ID` | Get a single severity. |
| `inc custom-fields list` | List custom fields. |
| `inc custom-fields show ID` | Get a single custom field. |
| `inc users list` | List users. Filter with `--email`. |
| `inc users show ID` | Get a single user. |
| `inc roles list` | List incident roles. |
| `inc roles show ID` | Get a single incident role. |
| `inc incident-updates list` | List incident updates. Filter with `--incident-id`. |
| `inc post-mortems list` | List post-mortem documents. Filter with `--incident-id`. |
| `inc post-mortems show ID` | Get a single post-mortem document. |
| `inc post-mortems content ID` | Get the full post-mortem content as markdown. |
| `inc follow-ups list` | List follow-up actions. Filter with `--incident-id`. |
| `inc follow-ups show ID` | Get a single follow-up. |
| `inc auth login` | Save your API key. |
| `inc auth status` | Check if authentication is working. |
| `inc auth token` | Print the current API key to stdout. |
| `inc config set KEY VALUE` | Set a config value. |
| `inc config get KEY` | Get a config value. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [incident-io/inc](https://github.com/incident-io/inc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
