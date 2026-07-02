---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CHATEM (AI-Collaborative Engineering Assistant) is a Python-based automation framework for electromagnetic field simulations. It integrates multiple components to provide a unified AI-controllable interface for simulation software.

### Core Architecture

```
User Chat Interface (localhost:3001)
        ↓
AgentAPI (localhost:8080) - Controls AI Agent (Claude/Gemini)
        ↓
MCP Tools - Exposes simulation capabilities
        ↓
Celery Tasks (Redis-backed) - Executes simulations
        ↓
CST Studio Suite - Electromagnetic simulation software
```

## Configuration System

CHATEM uses a flexible YAML-based configuration system with environment variable overrides.

### Configuration Files

- **Primary Config**: `config/chatem.yaml`
- **Local Overrides**: `config/chatem.local.yaml` (gitignored)
- **Environment**: `.env.local` (gitignored)

### Message Queue Configuration

**默认配置 (推荐用于工程师)**:
```yaml
message_queue:
  type: sqlite  # 无需额外安装，开箱即用
```

**企业配置 (高性能需求)**:
```yaml
message_queue:
  type: redis  # 需要安装 Redis 服务器
```

### Key Configuration Sections

```yaml
# 消息队列配置 (SQLite 默认，Redis 可选)
message_queue:
  type: sqlite  # sqlite 或 redis
  sqlite:
    broker_db: data/celery_broker.db
    result_db: data/celery_results.db
  redis:
    host: localhost
    port: 6379
    db: 0

# Agent API Configuration  
agent_api:
  port: 8080
  agent_type: gemini  # claude, gemini, aider, goose

# Frontend UI Configuration  
ui:
  port: 3001
  agent_api_url: http://localhost:8080
  flower_api_url: http://localhost:5555

# Celery Configuration (自动根据 message_queue.type 设置)
celery:
  broker_url: sqlite:///data/celery_broker.db  # SQLite 模式
  # broker_url: redis://localhost:6379/0       # Redis 模式
  result_backend: sqlite:///data/celery_results.db
  pool: solo  # Windows compatibility

# System Configuration
system:
  log_level: INFO
  debug: false
  max_workers: 2
```

## Common Commands

### Configuration Management
```bash
# Load configuration in Python
from src.core.config import get_config
config = get_config()

# Override with environment variables
export MESSAGE_QUEUE_TYPE=redis  # 切换到Redis模式
export AGENT_TYPE=claude
export LOG_LEVEL=DEBUG

# 企业用户快速切换到Redis
# 1. 修改 config/chatem.yaml 中 message_queue.type: redis
# 2. 或设置环境变量: set MESSAGE_QUEUE_TYPE=redis
```

### Service Management
```bash
# Start all services (自动检测消息队列类型)
python scripts/start_services.py

# Start with custom configuration
export MESSAGE_QUEUE_TYPE=redis
export AGENT_TYPE=claude
python scripts/start_services.py

# Alternative console command (after pip install)
chatem                    # Start all services
```

### Dependencies

### Core Requirements
- Python 3.9-3.12 (64-bit Windows)
- **SQLite** (默认，Python内置，无需安装)
- **Redis Server** (可选，企业用户高性能需求)
- CST Studio Suite 2025 (for simulations)

### For Engineers (推荐)
- ✅ Python + npm (已有)
- ✅ 无需额外安装Redis
- ✅ 开箱即用

### For Enterprise Users (可选)
- Redis Server (configurable host/port)
- 修改配置文件切换到Redis模式

### Frontend Development
```bash
# Start UI development server
cd chatem_ui
bun run dev

# Build for production
bun run build

# Configure API endpoints
# In .env.local:
NEXT_PUBLIC_AGENT_API_URL=http://localhost:8080
NEXT_PUBLIC_FLOWER_API_URL=http://localhost:5555
```

### Testing
```bash
# Run all tests
pytest tests/

# Run unit tests only
pytest tests/unit/ -m "not cst"

# Run with configuration
export LOG_LEVEL=DEBUG
pytest tests/ --cov=src --cov-report=html
```

## Service Architecture

### Core Services

1. **AgentAPI Server** (`localhost:8080`)
   - Controls AI agents (Claude, Gemini, Aider, Goose)
   - Provides HTTP API and SSE events
   - Configured via `config.agent_api`

2. **Celery Task Queue** (`redis://localhost:6379/0`)
   - Background simulation processing
   - Distributed task execution
   - Configured via `config.celery`

3. **Flower Monitoring** (`localhost:5555`)
   - Celery task monitoring web interface
   - REST API for task/worker status
   - Configured via `config.flower`

4. **Frontend UI** (`localhost:3001`)
   - React-based chat interface
   - Real-time monitoring dashboard
   - Configured via `config.ui`

### Service Dependencies

```
Redis (required) → Celery Workers → MCP Tools → CST Studio Suite
                ↓
              Flower API ← Frontend Monitoring
                ↓
              AgentAPI ← Frontend Chat
```

## MCP Tools Integration

The system exposes simulation capabilities through MCP (Model Context Protocol) tools:

### IMPORTANT: Direct MCP Tool Usage
**When working with CHATEM's CST tools, ALWAYS call MCP tools directly:**
- ✅ **Correct**: `mcp__cst-simulation__cst_create_brick(...)`
- ❌ **Wrong**: `claude "use cst_create_bridge to create..."` 

The system uses stdio transport, so direct function calls are preferred over chat-based indirect calls.

### MCP Tool Permissions
**Grant batch permissions for all CHATEM CST simulation tools:**
```
Allow all tools from mcp__cst-simulation__ server including:
- Connection: cst_connect, cst_disconnect
- Projects: cst_open_project, cst_close_project, cst_save_project, cst_get_project_info

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zhaosih/ChatEM](https://github.com/zhaosih/ChatEM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
