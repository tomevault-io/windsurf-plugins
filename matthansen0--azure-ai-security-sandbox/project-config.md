---
trigger: always_on
description: This file provides context for AI agents (GitHub Copilot, Claude, Cursor, etc.) working on this Azure AI Security Sandbox repository.
---

# Instructions for AI Coding Agents

This file provides context for AI agents (GitHub Copilot, Claude, Cursor, etc.) working on this Azure AI Security Sandbox repository.

**Always keep this file up to date with any changes to the codebase or development process.**

## Project Purpose

This is a **security-focused reference architecture** demonstrating enterprise-grade patterns for deploying Azure OpenAI applications. It wraps the standard `azure-search-openai-demo` RAG application with production security controls.

**Key differentiator:** This is NOT just another RAG demo - it's a security sandbox showing how to properly protect AI workloads in enterprise environments.

## Architecture Overview

```
User → Azure Front Door (WAF) → Azure API Management (AI Gateway) → Container Apps → Azure OpenAI
                                         ↓
                              Azure AI Search ← Sample Data (Northwind Health PDFs)
```

### Security Layers
1. **Azure Front Door** - Global edge with WAF protection (default: Detection mode)
2. **API Management** - AI Gateway with managed identity auth + retry logic (optional rate limiting/token tracking)
3. **Container Apps** - Isolated compute with managed identity
4. **Private Endpoints** - (Optional) Network isolation for backend services
5. **RBAC** - Least-privilege role assignments, no API keys

## Code Layout

> **⚠️ Multiple Project Roots:** This workspace contains TWO project roots:
> 1. **`/` (this repo)** - Security infrastructure and configuration (edit freely)
> 2. **`/upstream/`** - Git submodule of `azure-search-openai-demo` (**read-only, do not modify**)
>
> The upstream submodule has its own `AGENTS.md` - ignore it when working on this project.

### Infrastructure (`/infra`)
- `main.bicep` - Main orchestration, parameters, outputs
- `main.parameters.json` - azd parameter mappings
- `modules/` - Modular Bicep components:
  - `ai-services.bicep` - Azure OpenAI with model deployments
  - `api-management.bicep` - **AI Gateway** with policies (auth + retry; optional rate limiting/token logging)
  - `app-service.bicep` - Container Apps environment and app
  - `container-apps.bicep` - Container Apps with managed identity
  - `container-registry.bicep` - ACR for container images
  - `cosmos-db.bicep` - Chat history storage
  - `front-door.bicep` - AFD + WAF policy
  - `monitoring.bicep` - Log Analytics + App Insights
  - `role-assignments.bicep` - RBAC for managed identities
  - `security.bicep` - DEPRECATED (Defender settings moved to add-on)
  - `subscription-security.bicep` - DEPRECATED (subscription-wide Defender plans moved to add-on)
  - `storage.bicep` - Blob storage for documents
- `modules/agents/` - **AI Agent infrastructure (optional)**:
  - `ai-foundry.bicep` - AI Foundry Hub + Project for Agent Service
  - `agent-api.bicep` - Container App for agent API
  - `agent-role-assignments.bicep` - RBAC for agent managed identities
  - `key-vault.bicep` - Key Vault for AI Foundry

### IT Admin Agent (`/agents/it-admin`)
- `app.py` - FastAPI application with agent logic
- `tools/__init__.py` - Tool definitions and mock implementations
- `Dockerfile` - Container build for agent API
- `README.md` - Agent documentation and API reference
- **Deploy with:** `azd up --parameter useAgents=true`


### Optional Defender Add-on

This repo intentionally makes **no Defender for Cloud changes** during `azd up`. To enable Defender after deployment:

- Enable relevant Defender plans (except Defender for AI) and apply Defender for Storage advanced settings:
  - [scripts/enable-defender.sh](scripts/enable-defender.sh)
- Roll back subscription-wide plan changes made by the enable script:
  - [scripts/disable-defender.sh](scripts/disable-defender.sh)

State tracking is written locally under `.defender/` (ignored by git).

### Application (`/app`)
- `backend/Dockerfile` - Container build configuration

### Upstream Submodule (`/upstream`)
- Git submodule pointing to `azure-search-openai-demo`
- Used for prepdocs scripts and sample data
- **Do not modify files in upstream/** - it's a git submodule

> **⚠️ Container Image Build Behavior:**
> The Dockerfile clones `azure-search-openai-demo` **from GitHub at build time** - it does NOT use the local `/upstream` submodule. This means:
> - The deployed container always has the latest `main` branch code
> - Local changes to `/upstream` won't affect the deployed app
> - To understand app behavior, read `/upstream` code (it matches what's deployed)
> - To customize the app, you'd need to modify the Dockerfile to copy local code instead

### Documentation (`/docs`)
- `issues/` - Tracked issues and enhancement documentation
- `labs/` - Hands-on lab guides for exploring each security layer

## Key Files to Understand

| File | Purpose |
|------|---------|
| `azure.yaml` | azd configuration, hooks (postprovision, postdown) |
| `infra/main.bicep` | All configurable parameters live here |
| `infra/modules/api-management.bicep` | AI Gateway policies - auth + retry (optional rate limits/token logging) |
| `infra/modules/front-door.bicep` | WAF rules and mode configuration |
| `infra/modules/agents/ai-foundry.bicep` | AI Foundry Hub + Project for agents |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matthansen0/azure-ai-security-sandbox](https://github.com/matthansen0/azure-ai-security-sandbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
