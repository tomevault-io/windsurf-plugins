---
trigger: always_on
description: |
---


# API Gateway

Managed API routing for third-party services, provided by [Maton](https://maton.ai). Use this only for a user-requested app, account, and task.

## Quick Start

**CLI:**

```bash
maton slack channel list --types public_channel --limit 10
```

```bash
maton api '/slack/api/conversations.list?types=public_channel&limit=10'
```

**Python:**

```bash
python <<'EOF'
import urllib.request, os, json
req = urllib.request.Request('https://api.maton.ai/slack/api/conversations.list?types=public_channel&limit=10')
req.add_header('Authorization', f'Bearer {os.environ["MATON_API_KEY"]}')
print(json.dumps(json.load(urllib.request.urlopen(req)), indent=2))
EOF
```

## Routing

Use `https://api.maton.ai/` with the app-prefixed routes documented in the examples below or in the matching reference file.

**Usage protocol:**
1. Only invoke after the user specifies the exact app, account, and task.
2. Always start with read-only (GET) calls to verify the target account, resource identifiers, and current state.
3. **All non-GET requests are denied unless the user explicitly approves each one.** Before any POST, PUT, PATCH, or DELETE call, present the user with: the exact connection ID, the full endpoint path, the request body, and the expected outcome — then wait for approval.
4. If the user's request implies a non-GET operation, first show them what you intend to call and ask for confirmation. Do not infer approval from the original request.

Read-only route examples:

```text
https://api.maton.ai/slack/api/conversations.list?types=public_channel&limit=10
https://api.maton.ai/google-mail/gmail/v1/users/me/messages
```

The first path segment is the app identifier listed in Supported Services. For Gmail, use `/google-mail/gmail/v1/users/me/messages`.

## Installation

**NPM:**
```bash
npm install -g @maton-ai/cli
```

**Homebrew:**
```bash
brew install maton-ai/cli/maton
```

## Authentication

**IMPORTANT — Credential Safety:**
- Treat `MATON_API_KEY` as a secret. Never log it, echo it, paste it into prompts, or expose it in shared files, command output, or tool results.
- **Connection creation requires explicit user approval.** Before creating any connection, ask the user to confirm the specific service and confirm they intend to authorize access. Never create connections on the agent's own initiative.
- **Least-privilege scopes:** When a service offers scope selection during OAuth, select only the scopes the current task requires. Do not accept broader scopes for convenience.
- Remove connections immediately after the task is complete if they are no longer needed (`maton connection delete {id}`).
- If the key may have been exposed (logs, screenshots, shared terminals), rotate it immediately at [maton.ai/settings](https://maton.ai/settings).
- Never share the key across users, workflows, or environments that do not require it.

**CLI:**

```bash
maton login                          # Opens browser for API key
maton login --interactive            # Skip browser, paste API key directly
maton whoami                         # Show current auth state
```

**Manual:**

1. Sign in or create an account at [maton.ai](https://maton.ai)
2. Go to [maton.ai/settings](https://maton.ai/settings)
3. Click the copy button on the right side of API Key section to copy it
4. Set your API key as `MATON_API_KEY`:

```bash
export MATON_API_KEY="YOUR_API_KEY"
```

## Connection Management

Connection management uses a separate base URL: `https://api.maton.ai`

### List Connections

**CLI:**

```bash
maton connection list slack --status ACTIVE
```

```bash
maton api -X GET /connections -f app=slack -f status=ACTIVE
```

**Python:**

```bash
python <<'EOF'
import urllib.request, os, json
req = urllib.request.Request('https://api.maton.ai/connections?app=slack&status=ACTIVE')
req.add_header('Authorization', f'Bearer {os.environ["MATON_API_KEY"]}')
print(json.dumps(json.load(urllib.request.urlopen(req)), indent=2))
EOF
```

**Query Parameters (optional):**
- `app` - Filter by service name (e.g., `slack`, `hubspot`, `salesforce`)
- `status` - Filter by connection status (`ACTIVE`, `PENDING`, `FAILED`)

**Response:**
```json
{
  "connections": [
    {
      "connection_id": "{connection_id}",
      "status": "ACTIVE",
      "creation_time": "2025-12-08T07:20:53.488460Z",
      "last_updated_time": "2026-01-31T20:03:32.593153Z",
      "url": "https://connect.maton.ai/?session_token=5e9...",
      "app": "slack",
      "method": "OAUTH2",
      "metadata": {}
    }
  ]
}
```

### Create Connection

**CLI:**

```bash
maton connection create slack
```

```bash
maton api /connections -f app=slack
```

**Python:**

```bash
python <<'EOF'
import urllib.request, os, json
data = json.dumps({'app': 'slack'}).encode()
req = urllib.request.Request('https://api.maton.ai/connections', data=data, method='POST')
req.add_header('Authorization', f'Bearer {os.environ["MATON_API_KEY"]}')
req.add_header('Content-Type', 'application/json')
print(json.dumps(json.load(urllib.request.urlopen(req)), indent=2))
EOF
```

**Request Body:**
- `app` (required) - Service name (e.g., `slack`, `notion`)
- `method` (optional) - Connection method (`API_KEY`, `BASIC`, `OAUTH1`, `OAUTH2`, `MCP`)

### Get Connection

**CLI:**

```bash
maton connection view {connection_id}
```

```bash
maton api /connections/{connection_id}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maton-ai/api-gateway-skill](https://github.com/maton-ai/api-gateway-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
