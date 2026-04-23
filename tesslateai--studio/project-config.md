---
trigger: always_on
description: You are a senior level coding agent. You will apply real world solutions to all the problems, fixing them in such a way where you do not cheat the solution, break existing functionality, and are scoped in. The solutions you write must be scalable and for the future, not fixing or hardcoding.
---

You are a senior level coding agent. You will apply real world solutions to all the problems, fixing them in such a way where you do not cheat the solution, break existing functionality, and are scoped in. The solutions you write must be scalable and for the future, not fixing or hardcoding.

Always read through the docs/ to find items it is a knowledgegraph

Use subagents generously if you are doing bulk task items that have a small / atomic scope. 

don't do conditional logic for k8s and docker implementation differences. try to keep it as similar as possible unless if a platform requires differeces. Prioritize the k8s (keep that logic more intact than docker. )

On windows use MSYS_NO_PATHCONV=1 while running kubectl or docker exec commands. 
The ECR IS <AWS_ACCOUNT_ID> not <AWS_ACCOUNT_ID>

CRITICAL -- ENSURE ALL CHANGES ARE NON-BLOCKING

Everything u do or write should be non-blocking so certain actions don't hold up other people on our software. 

# Tesslate Studio

When I have an issue, fix it for the next time it happens in a general, scalable way. For example, if a container fails on startup, ensure all future container startups work 100%.

## What is Tesslate Studio?

AI-powered web application builder that lets users create, edit, deploy, and manage full-stack apps using natural language. Users describe what they want, an AI agent writes the code, and the platform handles containerized deployment.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    Tesslate Studio                          │
├─────────────────────────────────────────────────────────────┤
│  Frontend (app/)           │   Orchestrator (orchestrator/) │
│  React + Vite + TypeScript │   FastAPI + Python             │
│  - Monaco Editor           │   - Auth (JWT/OAuth)           │
│  - Live Preview            │   - Project Management         │
│  - Chat UI                 │   - AI Agent System            │
│  - File Browser            │   - Container Orchestration    │
├─────────────────────────────────────────────────────────────┤
│  Redis                │  ARQ Worker                          │
│  - Pub/Sub + Streams  │  - Distributed agent execution       │
│  - Task queue (ARQ)   │  - Progressive step persistence      │
│  - Distributed locks  │  - Webhook callbacks                 │
├─────────────────────────────────────────────────────────────┤
│  PostgreSQL        │  Docker/Kubernetes Container Manager   │
│  (User data,       │  (User project environments)           │
│   projects, chat)  │  - Per-project isolation               │
└─────────────────────────────────────────────────────────────┘
```

## Technology Stack

| Layer | Tech |
|-------|------|
| Frontend | React 19, TypeScript, Vite, Tailwind, Monaco Editor |
| Backend | FastAPI, Python 3.11, SQLAlchemy, LiteLLM |
| Database | PostgreSQL (asyncpg) |
| Task Queue | Redis 7.x, ARQ |
| Containers | Docker Compose (dev), Kubernetes (prod) |
| Routing | Traefik (Docker), NGINX Ingress (K8s) |
| AI | LiteLLM → OpenAI/Anthropic models |
| Payments | Stripe |

## Key Code Paths

### 1. Project Creation
```
POST /api/projects → routers/projects.py
  └─> _perform_project_setup (background task)
      ├─ Create project directory
      ├─ Copy template files from base
      ├─ Generate docker-compose.yml OR K8s manifests
      └─ Return project slug (e.g., "my-app-k3x8n2")
```

### 2. Agent Chat (AI Code Generation)
```
POST /api/chat/agent/stream → routers/chat.py
  ├─> Build AgentTaskPayload (agent_context.py)
  │     └─> Project info, git status, chat history, TESSLATE.md
  ├─> Enqueue to ARQ Redis queue
  │     └─> Worker picks up task (worker.py)
  │           ├─ Acquire project lock (prevent concurrent runs)
  │           ├─ Run agent loop with progressive persistence
  │           │   ├─ INSERT AgentStep per iteration
  │           │   ├─ Publish events to Redis Stream
  │           │   └─ Check cancellation signal between iterations
  │           ├─ Finalize Message with summary
  │           └─ Release lock + optional webhook callback
  └─> Redis Stream → WebSocket → Client renders steps in real-time
```

### 2b. External Agent API
```
POST /api/external/agent/invoke → routers/external_agent.py
  ├─> Authenticate via Bearer token (API key)
  ├─> Build AgentTaskPayload (same as browser flow)
  ├─> Enqueue to ARQ Redis queue
  └─> Return task_id + events_url immediately

GET /api/external/agent/events/{task_id} (SSE)
  └─> Subscribe to Redis Stream for real-time events

GET /api/external/agent/status/{task_id} (Polling)
  └─> Query TaskManager for current status
```

### 3. Container Lifecycle
```
POST /api/projects/{id}/start → routers/projects.py

DOCKER MODE (config.DEPLOYMENT_MODE="docker"):
  └─> DockerComposeOrchestrator.start_project()
      ├─ Generate docker-compose.yml from Container models
      ├─ docker-compose up -d
      ├─ Connect to Traefik network
      └─> URLs: {container}.localhost

KUBERNETES MODE (config.DEPLOYMENT_MODE="kubernetes"):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TesslateAI/Studio](https://github.com/TesslateAI/Studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
