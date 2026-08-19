---
trigger: always_on
description: - **FastAPI**: When reading, reviewing, editing, or creating ANY `.py` file that involves FastAPI (routes, middleware, lifespan, dependencies, Pydantic request/response models, or anything under `api/`), you MUST load the `fastapi` skill BEFORE doing any work. This is NON-NEGOTIABLE — never write or modify FastAPI code without the skill loaded. Skill location: [`~/.claude/skills/fastapi/SKILL.md`](file:///C:/Users/Ali/.claude/skills/fastapi/SKILL.md)
---

# CRM Digital FTE Factory

## Mandatory Skills

- **FastAPI**: When reading, reviewing, editing, or creating ANY `.py` file that involves FastAPI (routes, middleware, lifespan, dependencies, Pydantic request/response models, or anything under `api/`), you MUST load the `fastapi` skill BEFORE doing any work. This is NON-NEGOTIABLE — never write or modify FastAPI code without the skill loaded. Skill location: [`~/.claude/skills/fastapi/SKILL.md`](file:///C:/Users/Ali/.claude/skills/fastapi/SKILL.md)
- **OpenAI Agents SDK**: When reading, reviewing, editing, or creating ANY `.py` file that involves the OpenAI Agents SDK (`@function_tool`, `Agent`, `Runner`, `RunContextWrapper`, guardrails, handoffs, or anything under `agent/`), you MUST load the `openai-agents-sdk` skill BEFORE doing any work. This is NON-NEGOTIABLE — never write or modify agent code without the skill loaded. Skill location: [`~/.claude/skills/openai-agents-sdk/SKILL.md`](file:///C:/Users/Ali/.claude/skills/openai-agents-sdk/SKILL.md)
- **Next.js App Router**: When reading, reviewing, editing, or creating ANY `.tsx` or `.ts` file under `web/src/` (components, hooks, pages, layouts, routes, or anything in the `web/` directory), you MUST load the `nextjs-app-router` skill BEFORE doing any work. This is NON-NEGOTIABLE — never write or modify Next.js/React code without the skill loaded. Skill location: [`~/.claude/skills/nextjs-app-router/SKILL.md`](file:///C:/Users/Ali/.claude/skills/nextjs-app-router/SKILL.md)
- **Docker**: When reading, reviewing, editing, or creating ANY Dockerfile, docker-compose.yml, docker-compose.override.yml, .dockerignore, or Docker-related configuration file, you MUST load the `docker` skill BEFORE doing any work. This is NON-NEGOTIABLE — never write or modify Docker configurations without the skill loaded. Skill location: [`~/.claude/skills/docker/SKILL.md`](file:///C:/Users/Ali/.claude/skills/docker/SKILL.md)
- **Kubernetes**: When reading, reviewing, editing, or creating ANY Kubernetes manifest file (.yml/.yaml) under `k8s/` (Deployments, Services, ConfigMaps, Secrets, PVCs, HPAs, Namespaces, or any K8s resource), you MUST load the `kubernetes` skill BEFORE doing any work. This is NON-NEGOTIABLE — never write or modify K8s manifests without the skill loaded. Skill location: [`~/.claude/skills/kubernetes/SKILL.md`](file:///C:/Users/Ali/.claude/skills/kubernetes/SKILL.md)
- **Multiple skills**: If a file involves multiple technologies (e.g., `api/main.py` with FastAPI + OpenAI Agents SDK, or a `.tsx` file in a Next.js route that also calls agent APIs), you MUST load ALL matching skills before doing any work. Skills are independent conditions — if a file matches multiple skills, load ALL of them.

## Quick Reference

- **What:** 24/7 AI Customer Success agent handling Gmail, WhatsApp, and Web Form support
- **Tech Stack:** Python 3.12, FastAPI, OpenAI Agents SDK, PostgreSQL + pgvector, Redis
- **Full Spec:** [docs/hackathon-spec.md](docs/hackathon-spec.md) — all requirements, architecture, exercises, scoring, and deliverables live there. Read it before starting any work.

## Architecture

```
Request Flow:
  Client → POST /api/chat → 202 + job_id (instant)
                           → BackgroundTask: run_agent() → set_job() in Redis
  Client → GET /api/jobs/{id} → reads result from Redis

Sync Fallback:
  POST /api/chat?sync=true → 200 + response (blocks ~30s)
  Redis unavailable         → auto-falls back to sync mode

System Layers:
  api/main.py          → FastAPI HTTP layer (endpoints, Pydantic models)
  agent/               → AI agent (OpenAI Agents SDK, tools, context)
  agent/cache.py       → Redis caching + job store (get/set with graceful fallback)
  agent/tools/         → Function tools (customer, ticket, conversation, knowledge, response)
  database/            → PostgreSQL schema, migrations, seed data
  tests/               → pytest with fakeredis, async fixtures
```

## Key Files

| File | Purpose |
|------|---------|
| `api/main.py` | All endpoints, Pydantic models, background task helpers |
| `agent/cache.py` | Redis cache (KB search, channel config, customer lookup) + job store |
| `agent/context.py` | `AgentContext` dataclass (db_pool, openai_client, redis_client) |
| `agent/customer_success_agent.py` | Agent definition, system prompt, `run_agent()` |
| `agent/__init__.py` | Correlation ID (contextvars), JSON log formatter |
| `agent/tools/customer.py` | find_or_create_customer, get_customer_history, cross_channel_link |
| `agent/tools/ticket.py` | create_ticket, update_ticket, get_ticket |
| `agent/tools/conversation.py` | save_message, get_conversation_messages |
| `agent/tools/knowledge.py` | search_knowledge_base (OpenAI embedding + pgvector) |
| `agent/tools/response.py` | send_response (channel-aware, config-cached) |
| `tests/conftest.py` | Shared fixtures (mock_redis via fakeredis) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jawwad-ali/ai-customer-support-agent](https://github.com/jawwad-ali/ai-customer-support-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
