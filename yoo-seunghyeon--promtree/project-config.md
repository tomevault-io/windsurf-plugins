---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**S13P31S307**: Samsung Electronics Production Technology Research Institute collaboration project
**Goal**: Extract material property information from TDS/MSDS PDF files with 99%+ accuracy, store in databases, and enable RAG-based chatbot for material property prediction and Q&A.

### Team Structure

6-person team (Team PromTree):
- **RAG Development**: 양현지, 이석철, 전해지, 최인혁
- **Parser Development**: 유승현, 조은경

### Technology Stack

**Backend:**
- **Python 3.12**: Primary language
- **FastAPI**: Web API server
- **LangChain**: RAG framework
- **MongoDB**: Document storage (markdown, chunks)
- **PostgreSQL**: Structured property data storage
- **Elasticsearch**: Full-text search and indexing
- **Neo4j**: Knowledge graph storage
- **Qdrant**: Vector database
- **Ollama**: Local LLM inference
- **Google Gemini**: Cloud LLM API
- **Docker**: Database containerization

**Frontend:**
- **React 19.1**: UI framework
- **TypeScript**: Type-safe JavaScript
- **Vite**: Build tool and dev server
- **Tailwind CSS**: Utility-first CSS framework
- **Lucide React**: Icon library

## Repository Setup

- **Remote**: https://lab.ssafy.com/s13-final/S13P31S307.git
- **Main Branch**: `master`
- **Development Branch**: `develop`
- **Commit Convention**: `[S13P31S307-<issue-number>] <Type>: <description>` (Korean descriptions supported)
- **Branch Convention**: `S13P31S307-<issue-number>-<description>`

## Environment Setup

### Package Manager

This project uses **uv** for Python dependency management. Install uv if not already installed:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### Docker Services

Start all database services from the project root:

```bash
# Start all services
docker-compose up -d

# Or start specific services
docker-compose up -d mongodb qdrant
```

Services:
- **MongoDB** (port 27017): Document storage (no web UI)
- **Qdrant** (ports 6333, 6334): Vector database
- **Elasticsearch** (port 9200): Full-text search with Korean support (Nori)
- **Neo4j** (ports 7474, 7687): Knowledge graph database

Environment variables required in root `.env`:
```
MONGO_INITDB_ROOT_USERNAME=promtree
MONGO_INITDB_ROOT_PASSWORD=ssafy13s307
MONGO_HOST=localhost
MONGO_PORT=27017

# Google Gemini API for RAG
GOOGLE_API_KEY=your_api_key_here
```

**Note**: PostgreSQL, Mongo Express, Kibana, and Adminer are no longer in the default docker-compose. The project primarily uses MongoDB with Beanie ODM.

### Virtual Environment Setup

The project uses **uv** for unified dependency management via `pyproject.toml`. Individual modules may still have their own venvs for compatibility.

**Primary application** (using uv):
```bash
# Sync all dependencies from pyproject.toml
uv sync

# Activate the virtual environment
source .venv/bin/activate  # macOS/Linux
```

**Legacy modules** (if working on specific extraction pipelines):

**db3** (TDS property extraction):
```bash
cd db3
source venv/bin/activate
pip install -r requirements.txt
```

**retriever** (RAG system - legacy):
```bash
cd retriever
source venv/bin/activate
pip install -r rag_requirements.txt
```

## Quick Start (Full Stack Development)

To run the complete application (frontend + backend + databases):

### 1. Start Database Services

```bash
# From project root
docker-compose up -d
```

Verify services:
- MongoDB: Connect via `mongosh mongodb://promtree:ssafy13s307@localhost:27017`
- Qdrant: http://localhost:6333/dashboard
- Neo4j: http://localhost:7474 (browser UI, credentials: neo4j/ssafy13s307)
- Elasticsearch: `curl -u elastic:ssafy13s307 http://localhost:9200`

### 2. Start Backend API

```bash
# Install dependencies with uv (first time)
uv sync

# Activate virtual environment
source .venv/bin/activate  # macOS/Linux

# Configure environment (first time)
# Edit .env and add MONGO credentials + GOOGLE_API_KEY

# Start server (from project root)
python main.py
```

Backend will be available at:
- API: http://localhost:8000
- Swagger Docs: http://localhost:8000/docs
- Health Check: http://localhost:8000/

### 3. Start Frontend

```bash
cd frontend

# Install dependencies (first time)
npm install

# Start dev server
npm run dev
```

Frontend will be available at: http://localhost:5173

### Troubleshooting

**Port already in use:**
```bash
# Kill backend (port 8000)
lsof -ti:8000 | xargs kill -9

# Kill frontend (port 5173)
lsof -ti:5173 | xargs kill -9
```

**Database connection issues:**
```bash
# From project root
docker-compose restart mongodb
docker-compose ps  # Check status
docker logs mongodb  # Check MongoDB logs
```

**RAG initialization timeout:**
- First message query may take 30-60 seconds to initialize FAISS embeddings
- Subsequent queries will be faster

## Project Architecture

### High-Level Data Flow

```
PDF Files → PromTree → Markdown (MongoDB)
                ↓
         Property Extraction (db3 for TDS, db1/db1_2py for MSDS)
                ↓
         PostgreSQL Storage
                ↓
    Hybrid RAG System (retriever) → Chatbot UI
         ↓              ↓
    Vector Search   Graph Search
    (FAISS/Qdrant)  (Neo4j)
         ↓              ↓

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Yoo-SeungHyeon) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
