---
trigger: always_on
description: **AI Configuration Agent** is a Home Assistant add-on that provides an AI-powered assistant for managing Home Assistant configuration files through natural language. It uses OpenAI-compatible models to understand user requests, read configurations, propose changes, and apply them after user approval.
---

# CLAUDE.md - AI Configuration Agent for Home Assistant

## Project Overview

**AI Configuration Agent** is a Home Assistant add-on that provides an AI-powered assistant for managing Home Assistant configuration files through natural language. It uses OpenAI-compatible models to understand user requests, read configurations, propose changes, and apply them after user approval.

**Current Version:** 0.1.2

## Repository Information
- **GitHub:** https://github.com/yinzara/ha-config-ai-agent
- **License:** MIT
- **Primary Language:** Python 3.11+
- **Framework:** FastAPI with Uvicorn

---

## Architecture Summary

### Technology Stack
- **Backend:** FastAPI 0.109.0 (async web framework)
- **AI:** OpenAI Agents SDK (supports GPT-4, GPT-4o, OpenRouter, Ollama, Azure)
- **YAML:** ruamel.yaml 0.18.5 (comment-preserving parser)
- **WebSocket:** aiohttp 3.9.3 + websockets
- **Frontend:** Vanilla JavaScript with Marked.js, Diff.js
- **Deployment:** Docker container as Home Assistant add-on

### System Components

#### 1. Main Application (`src/main.py`)
- FastAPI application with Ingress support
- Health check endpoint: `/health`
- Configuration info: `/api/config`
- Chat endpoint: `/api/chat` (POST)
- Approval endpoint: `/api/approve` (POST)
- UI serving: `/` (index.html)

#### 2. Agent System (`src/agents/`)
- **agent_system.py:** Multi-agent orchestration using OpenAI function calling
- **tools.py:** AI tool functions for configuration operations
- **Configuration Agent:** Main agent that handles requests, reads configs, proposes changes
- **Tool Functions:**
  - `search_config_files`: Search and read all YAML files + virtual files (Lovelace, devices, entities, areas)
  - `propose_config_changes`: Batch file changes for approval workflow

#### 3. Configuration Manager (`src/config/manager.py`)
- Comment-preserving YAML operations (ruamel.yaml)
- Atomic file writes (write to temp, then move)
- Automatic backups with rotation (max 10 by default)
- Home Assistant validation via Supervisor API
- Automatic rollback on validation failure
- Path traversal protection
- Handles both real files and virtual files (devices, entities, areas, Lovelace)

#### 4. Home Assistant Integration (`src/ha_websocket.py`)
- WebSocket API client for HA Core
- Lovelace configuration retrieval and saving
- Device/Entity/Area registry access and updates
- Service call support
- Config reload triggers

#### 5. Workflow System (`src/workflow/`)
- Currently minimal (placeholder for Phase 4 enhancements)
- Approval workflow managed in AgentSystem
- Changeset storage with 1-hour expiration
- Diff generation in frontend

#### 6. User Interface (`templates/index.html`, `static/`)
- Dark-themed chat interface
- Real-time conversation with AI agent
- Approval cards with "View Changes" button
- Modal diff viewer with unified diff format
- Markdown rendering for assistant messages
- Loading indicators and system messages

---

## Key Features

### 1. Natural Language Configuration Management
- Ask questions about your configuration
- Request changes in plain English
- AI reads current config before proposing changes
- Explains the reasoning behind proposed changes

### 2. Safe Change Application
- All changes require explicit approval
- Visual diff before applying
- Automatic backups before each change
- Home Assistant validation before applying
- Automatic rollback if validation fails
- Supports batch changes to multiple files

### 3. Virtual File Support
- **Lovelace:** Read/write Lovelace dashboards via WebSocket API
- **Devices:** View/rename devices from device registry
- **Entities:** View/rename entities from entity registry
- **Areas:** View/create/update areas from area registry
- Virtual files appear as: `lovelace.yaml`, `devices/{id}.json`, `entities/{id}.json`, `areas/{id}.json`

### 4. Provider Flexibility
Configure any OpenAI-compatible API:
- **OpenAI:** GPT-5, GPT-5-mini, GPT-5-nano
- **Anthropic** Claude models
- **OpenRouter:** 100+ models
- **Local Ollama:** Privacy-first option
- **Azure OpenAI:** Enterprise deployments

### 5. Security Features
- Ingress authentication (Home Assistant login required)
- AppArmor container isolation
- Path traversal prevention
- Atomic file operations
- Validation before applying changes
- Automatic backups with rotation
- Manager-level Supervisor API access (required for validation)

---

## File Structure

```
ha-config-ai-agent/
├── config.yaml              # Add-on manifest (Ingress, options, permissions)
├── Dockerfile               # Container definition (Python 3.11-slim)
├── requirements.txt         # Python dependencies
├── run.sh                   # Startup script (reads options, sets env vars)
├── apparmor.txt            # Security profile
├── build.yaml              # Build configuration
│
├── src/
│   ├── main.py             # FastAPI application (315 lines)
│   ├── agents/
│   │   ├── agent_system.py # AI agent orchestration (454 lines)
│   │   └── tools.py        # Agent tool functions (989 lines)
│   ├── config/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yinzara/ha-config-ai-agent](https://github.com/yinzara/ha-config-ai-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
