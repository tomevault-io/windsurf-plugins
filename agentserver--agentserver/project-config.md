---
trigger: always_on
description: Endpoints under the `Agent` tag. Auto-generated from [`docs/api/openapi.yaml`](../openapi.yaml) — do not edit by hand.
---

# Agent

Endpoints under the `Agent` tag. Auto-generated from [`docs/api/openapi.yaml`](../openapi.yaml) — do not edit by hand.

> Run `make api-docs` after changing handler annotations to regenerate this file.

## Operations

| Method | Path | Summary |
|--------|------|---------|
| `GET` | [`/api/agent/discovery/agents`](#op-get-api-agent-discovery-agents) | Discover agents in the calling agent's workspace (proxy_token auth) |
| `POST` | [`/api/agent/discovery/cards`](#op-post-api-agent-discovery-cards) | Register or update an agent capability card |
| `GET` | [`/api/agent/mailbox/inbox`](#op-get-api-agent-mailbox-inbox) | Read messages from the calling agent's inbox |
| `POST` | [`/api/agent/mailbox/send`](#op-post-api-agent-mailbox-send) | Send a message to another agent's mailbox |
| `POST` | [`/api/agent/register`](#op-post-api-agent-register) | Register an agent (obtain sandbox credentials) |
| `POST` | [`/api/agent/tasks`](#op-post-api-agent-tasks) | Create a delegated task (proxy_token auth) |
| `GET` | [`/api/agent/tasks/poll`](#op-get-api-agent-tasks-poll) | Poll for pending tasks (proxy_token auth) |
| `GET` | [`/api/agent/tasks/{id}`](#op-get-api-agent-tasks-id) | Get a task by ID (proxy_token auth) |
| `PUT` | [`/api/agent/tasks/{id}/status`](#op-put-api-agent-tasks-id-status) | Update task status (proxy_token auth) |
| `GET` | [`/api/agent/whoami`](#op-get-api-agent-whoami) | Inspect the calling agent identity (proxy_token auth) |
| `GET` | [`/api/agents/{sandboxId}`](#op-get-api-agents-sandboxid) | Get a single agent card by sandbox ID |
| `GET` | [`/api/tasks/{id}`](#op-get-api-tasks-id) | Get a task by ID |
| `POST` | [`/api/tasks/{id}/cancel`](#op-post-api-tasks-id-cancel) | Cancel a task |
| `GET` | [`/api/workspaces/{wid}/agents`](#op-get-api-workspaces-wid-agents) | List agent cards in a workspace |
| `GET` | [`/api/workspaces/{wid}/tasks`](#op-get-api-workspaces-wid-tasks) | List delegated tasks for a workspace |
| `POST` | [`/api/workspaces/{wid}/tasks`](#op-post-api-workspaces-wid-tasks) | Create a delegated task in a workspace |

### `GET /api/agent/discovery/agents` {#op-get-api-agent-discovery-agents}
Discover agents in the calling agent's workspace (proxy_token auth)

**Responses**

| Status | Description | Schema |
|--------|-------------|--------|
| `200` | OK | array of [`AgentCardItem`](#schema-agentcarditem) |
| `401` | unauthorized | `string` |
| `500` | internal error | `string` |


### `POST /api/agent/discovery/cards` {#op-post-api-agent-discovery-cards}
Register or update an agent capability card

**Request body**

Content-Type: `application/json`

Schema: [`AgentCardRegisterRequest`](#schema-agentcardregisterrequest)

```yaml
{
  agent_type?: string
  card?: any
  description?: string
  display_name?: string
}
```


**Responses**

| Status | Description | Schema |
|--------|-------------|--------|
| `200` | OK | [`AgentCardRegisterResponse`](#schema-agentcardregisterresponse) |
| `400` | bad request | `string` |
| `401` | unauthorized | `string` |
| `500` | internal error | `string` |


### `GET /api/agent/mailbox/inbox` {#op-get-api-agent-mailbox-inbox}
Read messages from the calling agent's inbox

**Query parameters**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `limit` | `integer` | no | Max messages to return (default 10) |


**Responses**

| Status | Description | Schema |
|--------|-------------|--------|
| `200` | OK | array of [`AgentMailboxMessage`](#schema-agentmailboxmessage) |
| `401` | unauthorized | `string` |
| `500` | internal error | `string` |


### `POST /api/agent/mailbox/send` {#op-post-api-agent-mailbox-send}
Send a message to another agent's mailbox

**Request body**

Content-Type: `application/json`

Schema: [`AgentMailboxSendRequest`](#schema-agentmailboxsendrequest)

```yaml
{
  msg_type?: string
  text: string
  to: string
}
```


**Responses**

| Status | Description | Schema |
|--------|-------------|--------|
| `201` | Created | [`AgentMailboxSendResponse`](#schema-agentmailboxsendresponse) |
| `400` | bad request | `string` |
| `401` | unauthorized | `string` |
| `403` | target not in same workspace | `string` |
| `404` | target agent not found | `string` |
| `500` | internal error | `string` |


### `POST /api/agent/register` {#op-post-api-agent-register}
Register an agent (obtain sandbox credentials)

**Request body**

Content-Type: `application/json`

Schema: [`AgentRegisterRequest`](#schema-agentregisterrequest)

```yaml
{
  name?: string
  type?: string
}
```


**Responses**

| Status | Description | Schema |
|--------|-------------|--------|
| `201` | Created | [`AgentRegisterResponse`](#schema-agentregisterresponse) |
| `400` | bad request | `string` |
| `401` | unauthorized | `string` |
| `403` | no permission | `string` |
| `500` | internal error | `string` |


### `POST /api/agent/tasks` {#op-post-api-agent-tasks}
Create a delegated task (proxy_token auth)

**Request body**

Content-Type: `application/json`

Schema: [`AgentTaskCreateRequest`](#schema-agenttaskcreaterequest)

```yaml
{
  delegation_chain?: []string
  max_budget_usd?: number
  max_turns?: integer
  prompt: string
  requester_id?: string
  skill?: string

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentserver/agentserver](https://github.com/agentserver/agentserver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
