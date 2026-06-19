---
trigger: always_on
description: Interact with Mule AI workflow platform - manage providers, agents, skills, workflows, WASM modules, and execute AI tasks via OpenAI-compatible API.
---


# Mule AI Agent Skill

This skill helps you interact with a running Mule AI workflow platform. Mule is an AI workflow platform that enables you to create, configure, and execute AI agents and workflows through an OpenAI-compatible API.

## Prerequisites

### Server Connection

Before interacting with Mule, you must determine the server URL:

1. **Check for MULE_SERVER environment variable**:
   ```bash
   echo $MULE_SERVER
   ```

2. **If MULE_SERVER is not set**, ask the user to either:
   - Set the `MULE_SERVER` environment variable (e.g., `export MULE_SERVER=https://mule.butler.ooo`)
   - Provide the Mule server URL directly

3. **All API requests** should be made to the base URL stored in `MULE_SERVER` (e.g., `curl ${MULE_SERVER}/v1/models`)

### Base URL Variable

Throughout this skill, replace `${MULE_SERVER}` with the actual server URL. Common examples:
- `http://localhost:8080` (local development)
- `https://mule.butler.ooo` (production)

---

## Overview of Mule Primitives

Mule uses six core primitives stored in PostgreSQL:

1. **Providers** - AI provider configurations (OpenAI-compatible APIs)
2. **Skills** - Pi agent skills that can be assigned to agents
3. **Agents** - AI agents powered by pi RPC runtime
4. **WASM Modules** - WebAssembly modules for imperative code
5. **Workflows** - Ordered sequences of workflow steps
6. **Workflow Steps** - Individual execution steps (AGENT or WASM type)

---

## Listing Available Models

To see what agents and workflows are available, list the models:

```bash
curl -s ${MULE_SERVER}/v1/models | jq .
```

**Response format:**
```json
{
  "data": [
    { "id": "agent/my-agent", "object": "model", "owned_by": "mule" },
    { "id": "workflow/my-workflow", "object": "model", "owned_by": "mule" }
  ]
}
```

Agents are prefixed with `agent/` and workflows with `workflow/`.

---

## Executing Agents and Workflows

### Execute an Agent

Run an agent using the `/v1/chat/completions` endpoint:

```bash
curl -s -X POST ${MULE_SERVER}/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "agent/my-agent",
    "messages": [
      { "role": "user", "content": "Your prompt here" }
    ]
  }' | jq .
```

### Execute a Workflow (Synchronous)

Run a workflow synchronously (waits for completion):

```bash
curl -s -X POST ${MULE_SERVER}/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "workflow/my-workflow",
    "messages": [
      { "role": "user", "content": "Your input here" }
    ]
  }' | jq .
```

### Execute a Workflow (Asynchronous)

Run a workflow asynchronously (returns immediately with job ID):

```bash
curl -s -X POST ${MULE_SERVER}/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "async/workflow/my-workflow",
    "messages": [
      { "role": "user", "content": "Your input here" }
    ]
  }' | jq .
```

**Response includes job ID:**
```json
{
  "id": "job-uuid-here",
  "object": "async.job",
  "status": "queued",
  "message": "The workflow has been started"
}
```

### Check Job Status

For async workflows, check job status:

```bash
curl -s ${MULE_SERVER}/api/v1/jobs/{job-id} | jq .
```

### List All Jobs

```bash
curl -s "${MULE_SERVER}/api/v1/jobs?page=1&page_size=20" | jq .
```

Query parameters:
- `page` - Page number (default: 1)
- `page_size` - Results per page (default: 20, max: 100)
- `status` - Filter by status (queued, running, completed, failed)
- `search` - Search by job ID
- `workflow_name` - Filter by workflow name

---

## Managing Providers

Providers are AI provider configurations (OpenAI-compatible APIs).

### List Providers

```bash
curl -s ${MULE_SERVER}/api/v1/providers | jq .
```

### Create a Provider

```bash
curl -s -X POST ${MULE_SERVER}/api/v1/providers \
  -H "Content-Type: application/json" \
  -d '{
    "name": "openai",
    "api_base_url": "https://api.openai.com/v1",
    "api_key_encrypted": "your-encrypted-api-key"
  }' | jq .
```

### Get Provider Models

List available models for a provider:

```bash
curl -s ${MULE_SERVER}/api/v1/providers/{provider-id}/models | jq .
```

### Update a Provider

```bash
curl -s -X PUT ${MULE_SERVER}/api/v1/providers/{provider-id} \
  -H "Content-Type: application/json" \
  -d '{
    "name": "openai",
    "api_base_url": "https://api.openai.com/v1",
    "api_key_encrypted": "updated-key"
  }' | jq .
```

### Delete a Provider

```bash
curl -s -X DELETE ${MULE_SERVER}/api/v1/providers/{provider-id}
```

---

## Managing Skills

Skills are pi agent skills that can be assigned to agents. They define capabilities the agent can use.

### List Skills

```bash
curl -s ${MULE_SERVER}/api/v1/skills | jq .
```

### Create a Skill

```bash
curl -s -X POST ${MULE_SERVER}/api/v1/skills \
  -H "Content-Type: application/json" \
  -d '{
    "name": "web-search",
    "description": "Enables the agent to search the web",
    "path": "/path/to/skill/directory",
    "enabled": true
  }' | jq .
```

### Get a Skill

```bash
curl -s ${MULE_SERVER}/api/v1/skills/{skill-id} | jq .
```

### Update a Skill

```bash
curl -s -X PUT ${MULE_SERVER}/api/v1/skills/{skill-id} \
  -H "Content-Type: application/json" \
  -d '{

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mule-ai/mule](https://github.com/mule-ai/mule) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
