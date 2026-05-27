---
trigger: always_on
description: This document provides comprehensive guidelines for AI agents working on the Pixel Detective project, which consists of two production-ready applications:
---

# AI Agent Guidelines - Pixel Detective Project

## 🎯 **Overview**

This document provides comprehensive guidelines for AI agents working on the Pixel Detective project, which consists of two production-ready applications:

1. **Pixel Detective** - AI-powered media search engine
2. **Dev Graph** - Temporal knowledge graph for code evolution

**Key Principle**: This entire codebase was built through AI-assisted development. Every change should maintain this philosophy and code quality standards.

---

## 📋 **Quick Navigation Guide**

This root-level AGENTS.md serves as a **navigation hub**. For detailed development guidelines, consult the appropriate subdirectory documentation:

### **🎨 Pixel Detective (Media Search)**

**When working on:** Media search, image ingestion, CLIP/BLIP models, vector search, UMAP visualization

**Primary Documentation:**
- [`backend/AGENTS.md`](backend/AGENTS.md) - Backend service development (Ingestion, ML Inference, GPU-UMAP)
- [`frontend/AGENTS.md`](frontend/AGENTS.md) - Frontend development (Next.js, React, Chakra UI)
- [`backend/ARCHITECTURE.md`](backend/ARCHITECTURE.md) - Backend system design
- [`frontend/ARCHITECTURE.md`](frontend/ARCHITECTURE.md) - Frontend architecture

**Quick Reference:**
- Backend services: 3 FastAPI microservices (ports 8001, 8002, 8003)
- Frontend: Next.js 14 with App Router (port 3000)
- Database: Qdrant vector database (port 6333)
- AI Models: CLIP (embeddings), BLIP (captions), RAPIDS cuML (UMAP)

### **🗺️ Dev Graph (Code Knowledge Graph)**

**When working on:** Git history ingestion, knowledge graph, Neo4j, code symbol extraction, relationship derivation

**Primary Documentation:**
- [`developer_graph/AGENTS.md`](developer_graph/AGENTS.md) - Dev Graph module development
- [`developer_graph/architecture.md`](developer_graph/architecture.md) - System architecture and data model
- [`developer_graph/routes/AGENTS.md`](developer_graph/routes/AGENTS.md) - API route handlers
- [`dev_graph_audit/AGENTS.md`](dev_graph_audit/AGENTS.md) - Data quality and auditing

**Quick Reference:**
- Backend: FastAPI service (port 8080)
- Frontend: Next.js UI at `tools/dev-graph-ui` (port 3001)
- Database: Neo4j graph database (ports 7687/7474)
- Pipeline: 8-stage unified ingestion system

### **📚 Cross-Cutting Documentation**

**For project-wide concerns:**
- [`README.md`](README.md) - Project overview, quick start, features
- [`Architecture.MD`](Architecture.MD) - Legacy architecture reference
- [`docs/architecture.md`](docs/architecture.md) - Current system architecture for both apps
- [`DEVELOPER_GUIDE.md`](DEVELOPER_GUIDE.md) - Developer onboarding
- [`.cursor/rules/`](.cursor/rules/) - Context-specific coding guidelines

---

## 🏗️ **Repository Structure**

```
pixel-detective/
├── 🎨 Pixel Detective (Media Search)
│   ├── frontend/                          # Next.js UI (port 3000)
│   │   ├── src/app/                      # App router pages
│   │   ├── src/components/               # React components
│   │   ├── AGENTS.md                     # ← Frontend development guide
│   │   └── ARCHITECTURE.md
│   │
│   └── backend/                          # FastAPI services
│       ├── ingestion_orchestration_fastapi_app/  # Port 8002
│       ├── ml_inference_fastapi_app/              # Port 8001
│       ├── gpu_umap_service/                      # Port 8003
│       ├── AGENTS.md                     # ← Backend development guide
│       └── ARCHITECTURE.md
│
├── 🗺️ Dev Graph (Code Knowledge Graph)
│   ├── tools/dev-graph-ui/              # Next.js UI (port 3001)
│   │   └── src/
│   │
│   ├── developer_graph/                  # FastAPI API (port 8080)
│   │   ├── api.py                        # Main application
│   │   ├── routes/                       # API endpoints
│   │   │   └── AGENTS.md                # ← Route handler guide
│   │   ├── AGENTS.md                     # ← Dev Graph module guide
│   │   └── architecture.md               # ← Data model & pipeline
│   │
│   └── dev_graph_audit/                  # Analysis reports
│       └── AGENTS.md                     # ← Audit guide
│
├── 📚 Shared Resources
│   ├── docs/                            # Project documentation
│   │   ├── architecture.md              # ← System architecture (both apps)
│   │   ├── sprints/                     # Sprint planning & retrospectives
│   │   └── reference_guides/            # Technical guides
│   ├── .cursor/rules/                   # AI coding guidelines
│   │   ├── projectrules.mdc
│   │   ├── backend/                     # Backend-specific rules
│   │   └── frontend/                    # Frontend-specific rules
│   ├── utils/                           # Shared Python utilities
│   ├── database/                        # Database connectors
│   └── config.py                        # Global configuration
│
└── 🔧 DevOps & Scripts
    ├── docker-compose.yml               # Service orchestration
    ├── start_pixel_detective.{ps1,bat}  # Launch media search
    ├── start_dev_graph.{ps1,bat}        # Launch knowledge graph
    └── scripts/                         # Automation scripts
```

---

## 🚀 **Getting Started as an AI Agent**

### **Step 1: Identify Your Task Context**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rm2thaddeus/Pixel_Detective](https://github.com/rm2thaddeus/Pixel_Detective) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
