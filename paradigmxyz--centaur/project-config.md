---
trigger: always_on
description: Centaur runs locally on Kubernetes through the Helm chart. Infra secrets are required as pre-created Kubernetes Secrets. For local development, `just bootstrap-secrets` creates them from your shell environment:
---

# Centaur — Developer Guide

## Quick Start

### 1. Clone and configure

```bash
git clone <repo-url>
cd centaur
brew install just
```

Centaur runs locally on Kubernetes through the Helm chart. Infra secrets are required as pre-created Kubernetes Secrets. For local development, `just bootstrap-secrets` creates them from your shell environment:

```bash
export OP_SERVICE_ACCOUNT_TOKEN=...
export OP_VAULT=...
export SLACK_BOT_TOKEN=...
export SLACK_SIGNING_SECRET=...
export SLACKBOT_API_KEY=...
```

Application-level LLM/tool secrets such as OpenAI and Anthropic tokens stay in 1Password and are loaded by the secrets service.

### 2. Boot the stack

```bash
just up
```

### Database migrations

```bash
./scripts/dbmate new add_agent_leases
./scripts/dbmate --set overlay new add_org_tables
./scripts/dbmate status
./scripts/dbmate up
```

`./scripts/dbmate` creates the next numbered SQL file in `services/api/db/migrations` by default, or in `services/api/db/migrations` inside the mounted overlay when you pass `--set overlay`. `up`, `migrate`, and `status` run against both the core and overlay migration sets unless you pin a specific set. Each set has its own dbmate migrations table so overlay repos can extend the shared Postgres database without version collisions. If `DATABASE_URL` is not set in your shell, the wrapper reuses the API deployment's configured value through `kubectl exec`.

### 3. Test

From inside the API deployment (localhost bypass — no key needed):

```bash
THREAD_KEY=test-e2e-1

SPAWN=$(kubectl exec -n centaur deploy/centaur-centaur-api -- curl -s -X POST http://localhost:8000/agent/spawn \
  -H "Content-Type: application/json" \
  -d "{\"thread_key\":\"${THREAD_KEY}\",\"harness\":\"amp\"}")
ASSIGNMENT_GENERATION=$(printf '%s' "$SPAWN" | jq -r '.assignment_generation')

kubectl exec -n centaur deploy/centaur-centaur-api -- curl -s -X POST http://localhost:8000/agent/message \
  -H "Content-Type: application/json" \
  -d "{\"thread_key\":\"${THREAD_KEY}\",\"assignment_generation\":${ASSIGNMENT_GENERATION},\"role\":\"user\",\"parts\":[{\"type\":\"text\",\"text\":\"Reply with exactly PONG and nothing else.\"}]}"

EXECUTE=$(kubectl exec -n centaur deploy/centaur-centaur-api -- curl -s -X POST http://localhost:8000/agent/execute \
  -H "Content-Type: application/json" \
  -d "{\"thread_key\":\"${THREAD_KEY}\",\"assignment_generation\":${ASSIGNMENT_GENERATION},\"harness\":\"amp\",\"delivery\":{\"platform\":\"dev\"}}")
EXECUTION_ID=$(printf '%s' "$EXECUTE" | jq -r '.execution_id')

kubectl exec -n centaur deploy/centaur-centaur-api -- curl -s "http://localhost:8000/agent/executions/${EXECUTION_ID}" | jq
```

Or create a DB-backed key for external use (see [API Key Management](#api-key-management)).

## Architecture

See the [architecture diagram in the README](README.md#architecture).

### End-to-End Request Flow

1. User mentions bot in Slack → webhook → slackbot → api
2. API spawns/reuses a Kubernetes sandbox pod (`centaur-agent:latest`) for that thread
3. Executes harness (amp/claude-code/codex) through the sandbox backend
4. Harness calls tools via `curl` back to API at `http://api:8000` (REST, NOT MCP)
5. LLM API calls route through firewall proxy which injects real credentials
6. Results stream as JSON events → posted to Slack

### Service Interface Contracts

Centaur is a modular service architecture. Each service communicates through well-defined interfaces. As long as you implement these interfaces, you can swap or extend any layer independently.

**Client → API** (durable control-plane protocol):

Clients (slackbot, CLI, external integrations) should stay thin. They persist input with `spawn -> message -> execute`, stream or replay output from the durable events endpoint, and only fall back to durable terminal state when the live stream is gone. The API owns runtime assignment, execution serialization, cancellation, and final-delivery recovery; Postgres is the source of truth.

**Step 1: Assign or reuse a runtime** (`POST /agent/spawn`)

Pins one warm runtime to the thread and returns the current `assignment_generation`.

```
POST /agent/spawn
{
  "thread_key": "slack:C0AJ07U8Z1N:1773364194.179929",
  "harness": "amp"
}

← {
    "thread_key": "slack:C0AJ07U8Z1N:1773364194.179929",
    "runtime_id": "rtm_123",
    "assignment_generation": 12,
    "state": "assigned_idle"
  }
```

**Step 2: Persist the user turn** (`POST /agent/message`)

Writes one durable transcript event. Inline base64 image/document blocks are extracted into `attachments` and rewritten to lightweight `attachment_ref` parts.

```
POST /agent/message
{
  "thread_key": "slack:C0AJ07U8Z1N:1773364194.179929",
  "assignment_generation": 12,
  "role": "user",
  "parts": [{"type": "text", "text": "analyze this"}],
  "user_id": "U123",
  "metadata": {"user_name": "alice", "platform": "slack"}
}

← {"ok": true, "message_id": "msg_123"}
```

**Step 3: Enqueue execution** (`POST /agent/execute`)

Creates a durable execution request plus final-delivery obligation. The worker drives the attached container; the response is just the execution handle.

```
POST /agent/execute
{

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paradigmxyz/centaur](https://github.com/paradigmxyz/centaur) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
