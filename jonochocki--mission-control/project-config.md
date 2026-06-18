---
trigger: always_on
description: Interact with Mission Control — AI agent orchestration dashboard. Use when registering agents, managing tasks, syncing skills, or querying agent/task status via MC APIs.
---


# Mission Control Agent Skill

Mission Control (MC) is an AI agent orchestration dashboard with real-time SSE/WebSocket, a skill registry, framework adapters, and RBAC. This skill teaches agents how to interact with MC APIs programmatically.

## Quick Start

**Base URL:** `http://localhost:3000` (default Next.js dev) or your deployed host.

**Auth header:** `x-api-key: <your-api-key>`

**Register + heartbeat in two calls:**

```bash
# 1. Register
curl -X POST http://localhost:3000/api/adapters \
  -H "Content-Type: application/json" \
  -H "x-api-key: $MC_API_KEY" \
  -d '{
    "framework": "generic",
    "action": "register",
    "payload": { "agentId": "my-agent-01", "name": "My Agent" }
  }'

# 2. Heartbeat (repeat every 5 minutes)
curl -X POST http://localhost:3000/api/adapters \
  -H "Content-Type: application/json" \
  -H "x-api-key: $MC_API_KEY" \
  -d '{
    "framework": "generic",
    "action": "heartbeat",
    "payload": { "agentId": "my-agent-01", "status": "online" }
  }'
```

## Authentication

MC supports two auth methods:

| Method | Header | Use Case |
|--------|--------|----------|
| API Key | `x-api-key: <key>` or `Authorization: Bearer <key>` | Agents, scripts, CI/CD |
| Session cookie | `Cookie: __Host-mc-session=<token>` (HTTPS) or `mc-session=<token>` (HTTP) | Browser UI |

**Roles (hierarchical):** `viewer` < `operator` < `admin`

- **viewer** — Read-only access (GET endpoints)
- **operator** — Create/update agents, tasks, skills, use adapters
- **admin** — Full access including user management

API key auth grants `admin` role by default. The key is set via `API_KEY` env var or the `security.api_key` DB setting.

Agents can identify themselves with the optional `X-Agent-Name` header for attribution in audit logs.

## Agent Lifecycle

```
register → heartbeat (5m interval) → fetch assignments → report task status → disconnect
```

All lifecycle actions go through the adapter protocol (`POST /api/adapters`).

### 1. Register

```json
{
  "framework": "generic",
  "action": "register",
  "payload": {
    "agentId": "my-agent-01",
    "name": "My Agent",
    "metadata": { "version": "1.0", "capabilities": ["code", "review"] }
  }
}
```

### 2. Heartbeat

Send every ~5 minutes to stay marked as online.

```json
{
  "framework": "generic",
  "action": "heartbeat",
  "payload": {
    "agentId": "my-agent-01",
    "status": "online",
    "metrics": { "tasks_completed": 5, "uptime_seconds": 3600 }
  }
}
```

### 3. Fetch Assignments

Returns up to 5 pending tasks sorted by priority (critical → low), then due date.

```json
{
  "framework": "generic",
  "action": "assignments",
  "payload": { "agentId": "my-agent-01" }
}
```

**Response:**

```json
{
  "assignments": [
    { "taskId": "42", "description": "Fix login bug\nUsers cannot log in with SSO", "priority": 1 }
  ],
  "framework": "generic"
}
```

### 4. Report Task Progress

```json
{
  "framework": "generic",
  "action": "report",
  "payload": {
    "taskId": "42",
    "agentId": "my-agent-01",
    "progress": 75,
    "status": "in_progress",
    "output": "Fixed SSO handler, running tests..."
  }
}
```

`status` values: `in_progress`, `done`, `failed`, `blocked`

### 5. Disconnect

```json
{
  "framework": "generic",
  "action": "disconnect",
  "payload": { "agentId": "my-agent-01" }
}
```

## Core API Reference

### Agents — `/api/agents`

| Method | Min Role | Description |
|--------|----------|-------------|
| GET | viewer | List agents. Query: `?status=online&role=dev&limit=50&offset=0` |
| POST | operator | Create agent. Body: `{ name, role, status?, config?, template?, session_key?, soul_content? }` |
| PUT | operator | Update agent. Body: `{ name, status?, role?, config?, session_key?, soul_content?, last_activity? }` |

**GET response shape:**

```json
{
  "agents": [{
    "id": 1, "name": "scout", "role": "researcher", "status": "online",
    "config": {}, "taskStats": { "total": 10, "assigned": 2, "in_progress": 1, "completed": 7 }
  }],
  "total": 1, "page": 1, "limit": 50
}
```

### Tasks — `/api/tasks`

| Method | Min Role | Description |
|--------|----------|-------------|
| GET | viewer | List tasks. Query: `?status=in_progress&assigned_to=scout&priority=high&project_id=1&limit=50&offset=0` |
| POST | operator | Create task. Body: `{ title, description?, status?, priority?, assigned_to?, project_id?, tags?, metadata?, due_date?, estimated_hours? }` |
| PUT | operator | Bulk status update. Body: `{ tasks: [{ id, status }] }` |

**Priority values:** `critical`, `high`, `medium`, `low`

**Status values:** `inbox`, `assigned`, `in_progress`, `review`, `done`, `failed`, `blocked`, `cancelled`

Note: Moving a task to `done` via PUT requires an Aegis quality review approval.

**POST response:**

```json
{
  "task": {
    "id": 42, "title": "Fix login bug", "status": "assigned",
    "priority": "high", "assigned_to": "scout", "ticket_ref": "GEN-001",
    "tags": ["bug"], "metadata": {}
  }
}
```

### Skills — `/api/skills`

| Method | Min Role | Description |
|--------|----------|-------------|
| GET | viewer | List all skills across roots |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jonochocki/mission-control](https://github.com/jonochocki/mission-control) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
