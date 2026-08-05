---
trigger: always_on
description: Use 1000+ external apps via Composio - either directly through the CLI or by building AI agents and apps with the SDK
---


# composio

Use 1000+ external apps via Composio - either directly through the CLI or by building AI agents and apps with the SDK

## When to Apply
- User wants to access or interact with external apps (Gmail, Slack, GitHub, Notion, etc.)
- User wants to automate a task using an external service (send email, create issue, post message)
- Building an AI agent or app that integrates with external tools
- Multi-user apps that need per-user connections to external services

## Table of Contents

1. [Use Apps via Composio CLI](#use-apps-via-composio-cli)
   1.1. [Composio CLI Guide](#composio-cli-guide)

2. [Building Apps and Agents with Composio](#building-apps-and-agents-with-composio)
   2.1. [Building with Composio](#building-with-composio)

---

## 1. Use Apps via Composio CLI

<a name="use-apps-via-composio-cli"></a>

### 1.1. Composio CLI Guide

<a name="composio-cli-guide"></a>

**Impact:** 🟠 HIGH

> Use the Composio CLI to take actions on external apps directly - no code needed

# Composio CLI Guide

Use the Composio CLI to search, connect, and execute tools directly — no code writing required. Ideal for agents taking actions on behalf of the user.

## Prerequisites (first-time setup)

If the CLI is not installed or the user is not authenticated:

```bash
# Install
curl -fsSL https://composio.dev/install | bash
composio --version   # verify

# Authenticate
composio login       # OAuth flow; interactive org/project picker after login (use -y to skip)
composio whoami      # verify org_id, project_id, user_id (API keys are never displayed)
# Run upgrade in case you run into errors or starting with a new project
composio upgrade
```

> **Note**: Use `whoami` only to verify login status — do not hardcode these values in code. `whoami` shows hints for `composio manage orgs switch` and `composio init` when relevant.

**Login behavior**: By default, `composio login` shows an interactive org/project picker after OAuth. Use `composio login -y` to skip the picker and use session defaults. JSON output is emitted only after the picker finishes (or immediately with `-y`), so piped/scripted usage gets the correct `org_id` and `project_id`.

**Agent login (no direct browser access)**: For agents that cannot open a browser for the user, use a two-step flow:

1. **Get login URL and session key** — Print login URL and JSON (jq-parseable) then exit without opening browser or waiting:
   ```bash
   composio login --no-wait | jq
   ```
   Share the login URL with the agent's user to complete login in their browser.

2. **Complete login with session key** — Use the `key` from the previous output to check if the user has logged in:
   ```bash
   composio login --key "<key>"
   ```
   Without `--no-wait`: polls until the session is linked (same as browser flow after printing URL).
   With `--no-wait`: checks once and fails if not linked. For agents, once the user completes login, run with `--no-wait` to avoid blocking.

## Primary Workflow: search → link → execute

### Step 1 — Find the right tool

```bash
composio search "send an email"
composio search "create github issue"
composio search "post slack message"
```

The search results include connection status, so you can see immediately if the user is already connected to the required app.

> **Important**: Do not trim the output of `composio search` (e.g. with `head`). Use the full results to pick the right tool — truncating can hide the best match.

### Step 2 — Connect an account (if needed)

If the user is not connected to the app, link their account:

```bash
composio link gmail
composio link github
composio link slack
```

This opens an OAuth flow or prompts for credentials. Only needed once per app. By default, `composio link` waits until the connected account is ACTIVE (opens browser, polls). Use `--no-wait` for scripted or agent usage: it prints link info and JSON to stdout (JQ-friendly) and exits immediately. Output includes `status`, `connected_account_id`, `redirect_url`, and `toolkit`.

### Step 3 — Execute the tool

```bash
composio execute GMAIL_SEND_EMAIL --data '{"recipient_email":"you@example.com","subject":"Hello","body":"Test"}'
composio execute GITHUB_CREATE_AN_ISSUE --data '{"owner":"acme","repo":"my-repo","title":"Bug report"}'
```
To see a tool's input parameters before executing:
```bash
composio execute GMAIL_SEND_EMAIL --help
```

### Step 4 — Listen for events (optional)

```bash
composio listen
```

Streams real-time trigger events to the terminal.

---

## Tips for Agents

- **All commands output JSON** — pipe to `jq` for filtering and extraction
- **Agent login** — When the agent has no direct browser access, use `composio login --no-wait` to get the URL and key, share the URL with the user, then `composio login --key <key> --no-wait` once they complete login
- **Parallel execution** — use `&` and `wait` or shell scripts for complex multi-step tasks
- The default user context is the project's `test_user_id`. Pass `--user-id <id>` to act on behalf of a specific user.

```bash
composio execute GMAIL_SEND_EMAIL --user-id "user_123" --data '{"recipient_email":"them@example.com","subject":"Hi"}'
```

## Best Practices


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [melandlabs/openloomi](https://github.com/melandlabs/openloomi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
