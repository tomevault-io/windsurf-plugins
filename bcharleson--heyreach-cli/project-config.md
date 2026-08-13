---
trigger: always_on
description: > This file helps AI agents (GPT, Claude, Gemini, open-source models, etc.) install, authenticate, and use the HeyReach CLI to manage LinkedIn automation campaigns, leads, lists, conversations, and more via the HeyReach platform.
---

# AI Agent Guide — HeyReach CLI

> This file helps AI agents (GPT, Claude, Gemini, open-source models, etc.) install, authenticate, and use the HeyReach CLI to manage LinkedIn automation campaigns, leads, lists, conversations, and more via the HeyReach platform.

## Quick Start

```bash
# Install globally
npm install -g heyreach-cli

# Authenticate (non-interactive — best for agents)
export HEYREACH_API_KEY="your-api-key-here"

# Verify it works
heyreach status

# Or save credentials (validates key before saving)
heyreach login --api-key "your-api-key-here"
```

**Requirements:** Node.js 18+

## Authentication

Set your API key via environment variable — no interactive login needed:

```bash
export HEYREACH_API_KEY="your-api-key-here"
```

Or pass it per-command:

```bash
heyreach campaigns list --api-key "your-api-key-here"
```

Or save it permanently (validates the key first):

```bash
heyreach login --api-key "your-api-key-here"
# → {"success":true,"message":"Credentials saved and verified."}
```

API keys are generated from: HeyReach → Settings → Integrations → Public API

### Organization API (admin commands)

Organization commands (`heyreach org ...`) require a separate Organization API key:

```bash
export HEYREACH_ORG_API_KEY="your-org-api-key-here"
```

Or per-command: `heyreach org workspaces --org-key "your-org-key"`

### Key facts

- Auth header is `X-API-KEY` (not Bearer token)
- API keys never expire but can be deleted/deactivated
- Workspace key and Organization key are separate — each has its own 300 req/min rate limit
- Base URL is fixed: `https://api.heyreach.io/api/public/`
- `heyreach login` validates the key against the API before saving — invalid keys are rejected immediately

## Output Format

All commands output **JSON to stdout** by default — ready for parsing:

```bash
# Default: compact JSON
heyreach campaigns list
# → {"totalCount":5,"items":[{"id":123,"name":"Q1 Outreach","status":"IN_PROGRESS",...}]}

# Pretty-printed JSON
heyreach campaigns list --pretty

# Select specific fields
heyreach campaigns list --fields id,name,status

# Suppress output (exit code only)
heyreach campaigns list --quiet
```

**Exit codes:** 0 = success, 1 = error. Errors go to stderr as JSON:
```json
{"error":"No API key found. Run \"heyreach login\" or set HEYREACH_API_KEY.","code":"AUTH_ERROR"}
```

### Error codes

| Code | Meaning |
|------|---------|
| `AUTH_ERROR` | Missing or invalid API key |
| `NOT_FOUND` | Resource doesn't exist (bad ID, no matching lead, etc.) |
| `VALIDATION_ERROR` | Missing required fields or invalid input |
| `RATE_LIMIT` | 300 req/min exceeded (auto-retried with backoff) |
| `SERVER_ERROR` | HeyReach API error (auto-retried up to 3 times) |
| `HTTP_ERROR` | Other HTTP error |

## Discovering Commands

```bash
# List all command groups
heyreach --help

# List subcommands in a group
heyreach campaigns --help

# Get help for a specific subcommand (shows options + examples)
heyreach campaigns list --help
```

## Complete Command Reference

### campaigns (15 commands)
Manage LinkedIn outreach campaigns — including full programmatic creation, sequence design, scheduling, and activation so AI agents can build, launch, and run campaigns end-to-end without touching the UI.

| Command | Required flags | Optional flags | Description |
|---------|---------------|----------------|-------------|
| `list` | — | `--offset` `--limit` `--keyword` `--statuses` `--account-ids` | List campaigns (paginated) |
| `get` | `--campaign-id` | — | Get campaign by ID |
| `start` | `--campaign-id` | — | Activate a DRAFT campaign (DRAFT → IN_PROGRESS). Required after `create`; `resume` will reject DRAFTs |
| `resume` | `--campaign-id` | — | Resume a PAUSED / FINISHED / FAILED campaign. Will reject DRAFT campaigns — use `start` instead |
| `pause` | `--campaign-id` | — | Pause a running campaign. This is the only way to halt sending — there is no `delete`, `stop`, or `cancel` endpoint in the HeyReach API |
| `add-leads` | `--campaign-id` `--leads-json` | `--resume-finished` `--resume-paused` | Add leads (V2, returns counts) |
| `stop-lead` | `--campaign-id` | `--lead-member-id` `--lead-url` | Stop lead progression |
| `get-leads` | `--campaign-id` | `--offset` `--limit` `--time-from` `--time-to` `--time-filter` | Get leads with analytics |
| `get-for-lead` | at least one of: `--email` `--linkedin-id` `--profile-url` | `--offset` `--limit` | Find campaigns for a lead |
| `create` | `--name` `--list-id` `--account-ids` | `--schedule-json` `--sequence-json` `--exclude-list-id` `--exclude-contacted-other-campaigns` `--exclude-other-conversations` `--exclude-sender-contacted` | Create a fully configured campaign in DRAFT status (returns `{campaignId}`) |
| `update-settings` | `--campaign-id` `--name` `--list-id` | `--exclude-list-id` `--exclude-contacted-other-campaigns` `--exclude-other-conversations` `--exclude-sender-contacted` | Update campaign name, lead list, and exclusions. DRAFT/SCHEDULED/PAUSED only |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bcharleson/heyreach-cli](https://github.com/bcharleson/heyreach-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
