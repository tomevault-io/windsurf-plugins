---
trigger: always_on
description: Interact with Zendesk Support via CLI - search tickets, view details, analyze metrics, manage users/organizations, and update tickets. All responses are saved locally for efficient jq querying. All Zendesk content is screened for prompt injection (regex, semantic, and LLM-based) and wrapped with security markers before reaching the LLM.
---


# Zendesk CLI Skill

A command-line interface for comprehensive Zendesk API integration. Run commands via `uv run zd-cli <command>` in the skill directory.

> **Running commands**
>
> - **Installed (recommended):** `uvx zd-cli <command>`
> - **Development (cloned repo):** `uv run zd-cli <command>` from the repo directory
>
> Never use bare `python3 zd-cli` — dependencies won't be available. All examples below use `uv run zd-cli`; substitute `uvx zd-cli` if running outside the repo.

## Quick Start

```bash
# Test authentication
uv run zd-cli me

# Search tickets
uv run zd-cli search "status:open priority:urgent"

# Get ticket details
uv run zd-cli ticket-details 12345

# Query saved response (path shown in command output)
uv run zd-cli query <temp>/zd-cli-<UID>/ticket_details_xxx.json -q comments_slim
```

## Key Concepts

### Save First, Query Later

All API responses are automatically saved to `<temp>/zd-cli-<UID>/` (system temp directory) with:
- **Metadata**: Command, parameters, timestamp, item count
- **Structure**: Auto-extracted schema showing field types
- **Suggested queries**: Command-specific jq queries
- **Data**: Full API response

**Workflow pattern:**
1. Run a command (e.g., `uv run zd-cli ticket-details 12345`)
2. Response saved to a JSON file (path shown in output)
3. Use `uv run zd-cli query <file> -q <query_name>` to extract specific data
4. Avoid re-fetching - work with stored files

### Why This Matters

Zendesk API responses can be very large (comments with HTML, many custom fields). By saving locally and using jq:
- Context window is preserved (only extract what's needed)
- No redundant API calls
- Complex analysis can use multiple queries on same data

## Command Reference

### Ticket Commands

| Command | Description | Example |
|---------|-------------|---------|
| `search` | Search tickets with query | `uv run zd-cli search "status:open"` |
| `ticket` | Get ticket by ID | `uv run zd-cli ticket 12345` |
| `ticket-details` | Get ticket + all comments | `uv run zd-cli ticket-details 12345` |
| `linked-incidents` | Get incidents linked to problem | `uv run zd-cli linked-incidents 12345` |
| `attachment` | Download attachment file | `uv run zd-cli attachment --ticket 12345 <url>` |

### Write Operations

All write commands (`create-ticket`, `add-note`, `add-comment`) support **Markdown formatting** by default. Content is converted to HTML for reliable rendering in Zendesk Agent Workspace. Use `--plain-text` to send as plain text instead.

| Command | Description | Example |
|---------|-------------|---------|
| `update-ticket` | Update ticket properties | `uv run zd-cli update-ticket 12345 --status pending` |
| `create-ticket` | Create new ticket (Markdown) | `uv run zd-cli create-ticket "Subject" "**Bold** description"` |
| `add-note` | Add internal note (Markdown) | `uv run zd-cli add-note 12345 "**Investigation:** found the issue"` |
| `add-comment` | Add public comment (Markdown) | `uv run zd-cli add-comment 12345 "Here are the steps:\n- Step 1\n- Step 2"` |

### Metrics & Analytics

| Command | Description | Example |
|---------|-------------|---------|
| `ticket-metrics` | Get reply/resolution times | `uv run zd-cli ticket-metrics 12345` |
| `list-metrics` | List metrics for tickets | `uv run zd-cli list-metrics` |
| `satisfaction-ratings` | List CSAT ratings | `uv run zd-cli satisfaction-ratings --score bad` |
| `satisfaction-rating` | Get single rating | `uv run zd-cli satisfaction-rating 67890` |

### Views (Queue Management)

| Command | Description | Example |
|---------|-------------|---------|
| `views` | List available views | `uv run zd-cli views` |
| `view-count` | Get ticket count | `uv run zd-cli view-count 123` |
| `view-tickets` | Get tickets from view | `uv run zd-cli view-tickets 123` |

### Users & Organizations

| Command | Description | Example |
|---------|-------------|---------|
| `user` | Get user by ID | `uv run zd-cli user 12345` |
| `search-users` | Search users | `uv run zd-cli search-users "john@example.com"` |
| `org` | Get organization by ID | `uv run zd-cli org 67890` |
| `search-orgs` | Search organizations | `uv run zd-cli search-orgs "Acme"` |

### Authentication

| Command | Description | Example |
|---------|-------------|---------|
| `auth login` | Configure Zendesk API token credentials | `uv run zd-cli auth login` |
| `auth login-oauth` | OAuth 2.0 login (opens browser) | `uv run zd-cli auth login-oauth --subdomain co` |
| `auth status` | Check auth configuration (token + OAuth) | `uv run zd-cli auth status` |
| `auth logout` | Remove API token credentials | `uv run zd-cli auth logout` |
| `auth logout-oauth` | Remove OAuth token | `uv run zd-cli auth logout-oauth` |
| `auth login-slack` | Configure Slack webhook | `uv run zd-cli auth login-slack` |
| `auth status-slack` | Check Slack configuration | `uv run zd-cli auth status-slack` |
| `auth logout-slack` | Remove Slack configuration | `uv run zd-cli auth logout-slack` |

### Slack Integration


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andmarios/zendesk-skill](https://github.com/andmarios/zendesk-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
