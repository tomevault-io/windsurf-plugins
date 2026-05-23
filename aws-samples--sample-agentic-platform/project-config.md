---
trigger: always_on
description: | Change Type | Read This File |
---

# Agentic Platform - Agent Guide

## Quick Reference: Which AGENTS.md to Read

| Change Type | Read This File |
|-------------|----------------|
| **Agent/Service code** | [`src/agentic_platform/AGENTS.md`](src/agentic_platform/AGENTS.md) |
| **Infrastructure (Terraform)** | [`infrastructure/AGENTS.md`](infrastructure/AGENTS.md) |
| **Kubernetes/Helm** | [`k8s/AGENTS.md`](k8s/AGENTS.md) |
| **Bootstrap/CloudFormation** | [`bootstrap/AGENTS.md`](bootstrap/AGENTS.md) |
| **Tests** | [`tests/AGENTS.md`](tests/AGENTS.md) |
| **Labs (learning only)** | [`labs/AGENTS.md`](labs/AGENTS.md) |

## Critical Rules (All Changes)

```bash
# After code changes
make test

# After EVERY commit
make security

# After Terraform changes
checkov -d .
```

## Make Commands

Run `make help` for all available commands. Key commands:

```bash
# Setup
make install              # Install dependencies

# Testing
make test                 # Run all tests
make test-unit            # Run unit tests only
make test-cov             # Run tests with coverage

# Run locally (agents)
make dev agentic_chat              # Run an agent locally
make dev agentic_rag PORT=8004     # Run with custom port

# Run locally (MCP servers)
make dev:mcp bedrock_kb_mcp_server # Run an MCP server locally

# Run locally (services)
make service memory_gateway        # Run a service locally

# Build & Deploy (agents)
make build agentic-chat              # Build container
make deploy-eks agentic-chat         # Build + deploy to EKS
make deploy-ac agentic_chat          # Build + deploy to AgentCore

# Build & Deploy (MCP servers)
make build:mcp bedrock-kb-mcp-server       # Build MCP container
make deploy-eks:mcp bedrock-kb-mcp-server  # Build + deploy MCP to EKS

# Code quality
make lint                 # Run linter
make security             # Run gitleaks
```

---

# Agent Development Guide

This guide explains the agent architecture, folder structure, and how to build new agents in the platform.

## Table of Contents

- [Architecture Overview](#architecture-overview)
- [Folder Structure](#folder-structure)
- [Agent Patterns](#agent-patterns)
- [Building a New Agent](#building-a-new-agent)
- [Core Components](#core-components)
- [Deployment](#deployment)
- [Testing](#testing)

## Architecture Overview

The platform uses a microservice architecture where each agent runs as an independent FastAPI server. Agents share a common core package that provides:

- **Standardized API models** (`AgenticRequest`, `AgenticResponse`)
- **Gateway clients** (LLM, Memory, Retrieval)
- **Middleware** (Authentication, Telemetry, Request Context)
- **Observability** (OpenTelemetry integration)
- **Converters** (Framework-specific adapters)

### Key Principles

1. **Separation of Concerns**: Server → Controller → Agent logic
2. **Framework Agnostic**: Support multiple agent frameworks (LangGraph, PydanticAI, Strands, DIY)
3. **Secure by Design**: JWT authentication, no direct IAM roles on agents
4. **Observable**: Built-in telemetry and tracing
5. **Containerized**: Each agent is independently deployable

## Folder Structure

```
src/agentic_platform/
├── agent/                          # Agent implementations
│   ├── agentic_chat/               # Strands-based chat agent
│   │   ├── server.py               # FastAPI server (thin layer)
│   │   ├── controller/             # Business logic
│   │   │   └── agentic_chat_controller.py
│   │   ├── agent/                  # Agent implementation
│   │   │   └── agentic_chat_agent.py
│   │   ├── prompt/                 # Prompt templates
│   │   │   └── agentic_chat_prompt.py
│   │   ├── streaming/              # Streaming utilities
│   │   │   └── strands_converter.py
│   │   ├── Dockerfile              # Container definition
│   │   ├── requirements.txt        # Dependencies
│   │   └── .env                    # Local config
│   │
│   ├── agentic_rag/                # RAG agent with Bedrock KB
│   │   ├── server.py
│   │   ├── controller/
│   │   ├── agent/
│   │   ├── tool/                   # Agent-specific tools
│   │   ├── prompt/
│   │   ├── Dockerfile
│   │   ├── requirements.txt
│   │   └── .env
│   │
│   ├── langgraph_chat/             # LangGraph-based agent
│   │   ├── server.py
│   │   ├── chat_controller.py
│   │   ├── chat_workflow.py        # LangGraph workflow
│   │   ├── chat_prompt.py
│   │   ├── Dockerfile
│   │   ├── requirements.txt
│   │   └── .env
│   │
│   ├── jira_agent/                 # Jira integration agent
│   │   ├── server.py
│   │   ├── jira_controller.py
│   │   ├── jira_agent.py
│   │   ├── jira_prompt.py
│   │   ├── Dockerfile
│   │   ├── requirements.txt
│   │   └── .env
│   │
│   └── strands_glue_athena/        # AWS Glue/Athena agent
│       ├── server.py
│       ├── agent_controller.py
│       ├── agent_service.py
│       ├── tools/                  # AWS-specific tools
│       │   ├── athena_tools.py
│       │   └── glue_tools.py
│       ├── Dockerfile
│       ├── requirements.txt
│       └── README.md
│
├── core/                           # Shared core functionality
│   ├── client/                     # Gateway clients
│   │   ├── llm_gateway/
│   │   ├── memory_gateway/
│   │   └── retrieval_gateway/
│   ├── models/                     # Shared data models
│   │   ├── api_models.py           # AgenticRequest/Response

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws-samples/sample-agentic-platform](https://github.com/aws-samples/sample-agentic-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
