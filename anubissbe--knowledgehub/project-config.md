---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 🧠 Project Overview

**KnowledgeHub** is an enterprise AI-enhanced development platform that provides persistent memory, advanced knowledge systems, and intelligent automation for AI coding assistants. It's designed as a universal backend that works with ANY AI coding tool (Claude Code, VSCode Copilot, Cursor, etc.) via REST APIs, WebSocket, and MCP (Model Context Protocol) integration.

### Core Value Proposition
- **Universal Backend**: Works with ANY AI coding assistant, not tool-specific
- **Persistent Memory**: Session continuity and context across all your tools
- **Learning System**: Continuously improves from usage patterns and mistakes
- **Enterprise Ready**: Multi-tenant, GDPR compliant, 99.9% uptime SLA
- **Production Tested**: 90%+ test coverage, comprehensive monitoring

## 🏗️ Architecture Overview

### Microservices Stack
```
┌─────────────────────────────────────────────────────────────────┐
│                    AI Coding Tools                              │
│        (Claude Code, Cursor, Copilot, Codeium, etc)           │
└──────────────────────────┬──────────────────────────────────────┘
                          │ REST API / WebSocket / MCP
┌──────────────────────────┴──────────────────────────────────────┐
│                    KnowledgeHub API Gateway                     │
│                      FastAPI (Port 3000)                       │
├─────────────────────────────────────────────────────────────────┤
│ 8 AI Intelligence Systems + Enterprise Features                │
│ • Session Continuity     • Mistake Learning    • Decision AI   │
│ • Code Evolution        • Performance Intel    • Workflow AI   │
│ • GraphRAG + Neo4j      • LlamaIndex RAG      • Real-time AI  │
└────┬────────────┬────────────┬────────────┬────────────────────┘
     │            │            │            │
┌────┴────┐ ┌────┴────┐ ┌────┴────┐ ┌────┴────┐
│PostgreSQL│ │TimescaleDB│ │  Neo4j  │ │ Weaviate│
│(Primary) │ │(Analytics)│ │ (Graph) │ │(Vectors)│
│   +      │ │     +     │ │    +    │ │    +    │
│  Redis   │ │   MinIO   │ │Prometheus│ │ Grafana │
└─────────┘ └─────────────┘ └─────────┘ └─────────┘
```

### Database Architecture
| Database | Purpose | Performance | Scale |
|----------|---------|-------------|--------|
| **PostgreSQL** | Primary data, sessions, metadata | 10K+ req/sec | Multi-TB |
| **TimescaleDB** | Time-series analytics, metrics | 1M+ points/sec | Hypertables |
| **Neo4j** | Knowledge graphs, relationships | Sub-100ms traversal | 100M+ nodes |
| **Weaviate** | Vector embeddings, semantic search | 1K+ vectors/sec | Billions |
| **Redis** | Cache, sessions, real-time data | 100K+ ops/sec | Memory-optimized |
| **MinIO** | Object storage, files, backups | S3-compatible | Petabyte+ |

### Core AI Intelligence Systems
1. **Session Continuity**: Auto-restore sessions with full context across tools
2. **Mistake Learning**: ML-powered error pattern recognition and prevention
3. **Decision Reasoning**: Track technical decisions with alternatives and outcomes
4. **Proactive Assistant**: Predict next tasks based on development patterns
5. **Code Evolution**: Track code changes with context and reasoning
6. **Performance Intelligence**: Monitor and optimize development workflows
7. **Workflow Automation**: Pattern-based automation and template generation
8. **Advanced Analytics**: Productivity metrics and project health insights

## 🚀 Development Setup

### Prerequisites
- Python 3.11+
- Node.js 18+ (for frontend)
- Docker & Docker Compose
- 8GB+ RAM recommended
- 10GB+ free disk space

### Quick Start
```bash
# Clone and start all services
git clone <repository>
cd knowledgehub

# Start all services (PostgreSQL, Redis, Weaviate, Neo4j, TimescaleDB, MinIO, API, Frontend)
docker-compose up -d

# Verify all services are healthy
docker-compose ps --filter "status=running"

# Check API health
curl http://localhost:3000/health

# Access web interface
open http://localhost:3100
```

### Service Endpoints
- **Main API**: http://localhost:3000 (FastAPI with OpenAPI docs at /docs)
- **Web UI**: http://localhost:3100 (React frontend)
- **PostgreSQL**: localhost:5433 (knowledgehub/knowledgehub123)
- **TimescaleDB**: localhost:5434 (analytics)
- **Redis**: localhost:6381
- **Weaviate**: localhost:8090
- **Neo4j**: localhost:7474 (web) / 7687 (bolt)
- **MinIO**: localhost:9010 (API) / 9011 (console)

## 💻 Common Development Commands

### Backend Development (Python/FastAPI)
```bash
# Install dependencies
pip install -r requirements.txt

# Run API locally (development)
cd api && python -m uvicorn main:app --reload --host 0.0.0.0 --port 3000

# Code quality checks
black . --line-length 88                    # Code formatting
flake8 . --max-line-length=88                # Linting
mypy . --ignore-missing-imports              # Type checking

# Testing
pytest                                       # Run all tests
pytest -m unit                              # Run unit tests only
pytest -m integration                       # Run integration tests
pytest -m e2e                              # Run end-to-end tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anubissbe/knowledgehub](https://github.com/anubissbe/knowledgehub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
