---
trigger: always_on
description: This file provides comprehensive guidance to Claude Code when working with the RAGViz platform - a Graph RAG Visualization Platform that combines knowledge graphs with vector retrieval for local LLM applications.
---

# CLAUDE.md - RAGViz Development Guide

This file provides comprehensive guidance to Claude Code when working with the RAGViz platform - a Graph RAG Visualization Platform that combines knowledge graphs with vector retrieval for local LLM applications.

## 🎯 Project Overview

**RAGViz** is a developer-oriented Graph Retrieval-Augmented Generation (Graph RAG) visualization and parameter tuning platform. It combines knowledge graphs with vector retrieval to provide contextual support for local large language models, enabling interactive document uploads, knowledge graph construction, and real-time parameter adjustments while visualizing their impact on answers and subgraph structures.

### Core Philosophy

- **KISS (Keep It Simple, Stupid)**: Choose straightforward solutions over complex ones
- **YAGNI (You Aren't Gonna Need It)**: Implement features only when needed
- **Developer-First**: Built for research and data science teams with debugging visualization
- **Local-First**: Privacy-focused with local LLM integration (Ollama/llama.cpp)

## 🏗️ Architecture Overview

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │    Backend      │    │   Database      │
│   (Next.js)     │◄──►│   (FastAPI)     │◄──►│   (Neo4j)       │
│                 │    │                 │    │                 │
│ • File Upload   │    │ • Document Proc │    │ • Knowledge     │
│ • Chat UI       │    │ • Entity Extract│    │   Graph         │
│ • Graph Viz     │    │ • Vector Search │    │ • Vector Index  │
│ • Parameters    │    │ • LLM Inference │    │ • HNSW Search   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                ▲
                                │
                       ┌─────────────────┐
                       │   Local LLM     │
                       │ (Ollama/llama.cpp)│
                       └─────────────────┘
```

**Data Flow**:
1. User uploads documents → Frontend → Backend
2. Backend processes: chunking, entity extraction, embedding → Neo4j
3. User queries via chat → Backend retrieves via vector search → LLM generates response
4. Frontend displays answer + highlights retrieved subgraph

## 📁 Project Structure

```
ragviz/
├── frontend/                 # Next.js React Application
│   ├── src/
│   │   ├── app/             # Next.js App Router pages
│   │   ├── components/      # React components
│   │   │   ├── graph/       # Graph visualization components
│   │   │   ├── ui/          # Reusable UI components (shadcn/ui)
│   │   │   └── chat/        # Chat interface components
│   │   └── lib/             # Utilities and API clients
│   ├── package.json         # Node.js dependencies
│   ├── next.config.ts       # Next.js configuration
│   └── CLAUDE.md           # Frontend-specific guide
├── backend/                 # FastAPI Python Application
│   ├── app/
│   │   ├── api/            # API route handlers
│   │   ├── core/           # Configuration and settings
│   │   ├── services/       # Business logic (Neo4j, LLM, etc.)
│   │   └── models/         # Pydantic data models
│   ├── requirements.txt    # Python dependencies
│   ├── venv/              # Virtual environment
│   └── CLAUDE.md          # Backend-specific guide (detailed)
├── docker-compose.yml      # Container orchestration
├── .env.example           # Environment variables template
├── README.md              # Project documentation
├── PROJECT.md             # Detailed technical overview (Chinese)
└── CLAUDE.md              # This comprehensive guide
```

## 🛠️ Development Environment Setup

### Prerequisites

- **Node.js** 18+ (for frontend)
- **Python** 3.9+ (for backend)
- **Neo4j** 5.15+ (database) - Choose one:
  - **Recommended**: [Neo4j Aura Free](https://neo4j.com/cloud/aura-free/) (cloud, zero setup)
  - Local: Docker, Neo4j Desktop, or self-hosted
- **Ollama** (for local LLM)
- **Docker** & Docker Compose (optional, for all-in-one setup)

### Quick Start

#### Option 1: Cloud Setup with Neo4j Aura (Recommended) ☁️

Best for beginners - no local database installation needed!

```bash
# 1. Get Neo4j Aura Free (2 minutes)
# Visit: https://neo4j.com/cloud/aura-free/
# Create free instance, save credentials

# 2. Clone and setup
git clone <repository-url>
cd ragviz

# 3. Environment setup
cp .env.example .env
# Edit .env:
# NEO4J_URI=neo4j+s://xxxxx.databases.neo4j.io  # Your Aura URI
# NEO4J_USERNAME=neo4j
# NEO4J_PASSWORD=your_aura_password

# 4. Install Ollama
# Visit https://ollama.ai/download
ollama pull qwen2.5:7b-instruct

# 5. Run Backend
cd backend
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000

# 6. Run Frontend (new terminal)
cd frontend
npm install
npm run dev  # Uses Turbopack for fast development (2s vs 30s+)
```

#### Option 2: Docker (All-in-One Local Setup) 🐳

```bash
# 1. Clone and setup
git clone <repository-url>
cd ragviz

# 2. Environment setup
cp .env.example .env
# Edit .env with local settings (already configured for Docker)

# 3. Start everything
docker-compose up -d


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zhijiewong/ragviz](https://github.com/zhijiewong/ragviz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
