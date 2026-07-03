---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Hybrid Search RAG for Academic Research Papers** - A production-ready hybrid search RAG system combining BM25 (lexical) and semantic search with advanced chunking strategies, optimized for academic research papers.

**Status:** Early development phase - project structure not yet created
**Package Manager:** UV (Python 3.12)
**Timeline:** 12-week development (6 weeks MVP + 6 weeks enhancements)

## Technology Stack

### Core Technologies
- **Backend:** FastAPI with asyncio/uvicorn
- **Parser:** Docling (PDF, HTML, DOCX)
- **Chunking:** Chonkie (SemanticChunker, TokenChunker, SDPMChunker)
- **Embeddings:** BGE (BAAI/bge-base-en-v1.5, 768 dimensions)
- **Vector DB:** Qdrant (semantic search)
- **Search:** Elasticsearch (BM25 lexical search)
- **Reranker:** Cohere rerank-english-v3.0
- **LLM:** Anthropic Claude (primary), OpenAI GPT-4 (fallback)

### Infrastructure
- **Cloud:** AWS (US-East-1)
- **Storage:** S3 (documents, logs, artifacts)
- **Caching:** Redis
- **Container:** Docker, Amazon ECR
- **Orchestration:** Kubernetes (Amazon EKS) - Post-MVP
- **Observability:** Loguru (structured JSON logging), OPIK (tracing), Comet ML (experiments)

## Development Commands

### Environment Setup
```bash
# Install UV package manager (if not installed)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Create virtual environment and install dependencies
uv sync

# Activate virtual environment
source .venv/bin/activate
```

### Local Services (Docker Compose)
```bash
# Start all local services (Qdrant, Elasticsearch, Redis)
docker-compose -f docker/docker-compose.yml up -d

# Stop services
docker-compose -f docker/docker-compose.yml down

# View logs
docker-compose -f docker/docker-compose.yml logs -f
```

### Running the API
```bash
# Development mode with hot reload
uvicorn src.api.main:app --reload --host 0.0.0.0 --port 8000

# Production mode
uvicorn src.api.main:app --host 0.0.0.0 --port 8000 --workers 4
```

### Testing
```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=src --cov-report=html --cov-report=term

# Run specific test file
pytest tests/unit/test_chunking.py

# Run integration tests only
pytest tests/integration/

# Run with verbose output
pytest -v -s
```

### Code Quality
```bash
# Format code
ruff format .

# Lint code
ruff check .

# Type checking
mypy src/
```

### Load Testing
```bash
# Run Locust load tests
locust -f tests/load/locustfile.py --host=http://localhost:8000
```

## Architecture

### Document Processing Pipeline
```
File Upload → S3 (raw) → Format Detection → Parser Selection →
Text Extraction → Metadata Extraction → Preprocessing →
Quality Validation → S3 (parsed) → Chunking → Dual Indexing
```

### Chunking Strategy
The system implements three chunking approaches via Chonkie:
1. **TokenChunker:** Baseline fixed-token chunking
2. **SemanticChunker:** Primary method for standard documents
3. **SDPMChunker:** Advanced method for long academic papers (>10k tokens)

**Academic-specific chunking:**
- Section-aware: Preserves paper structure (Abstract, Methods, Results, etc.)
- Equation-aware: Keeps LaTeX equations intact with context
- Reference-aware: Maintains citation context
- Abstract handling: Stored as a single complete chunk

### Dual Indexing Architecture
Documents are indexed in parallel:
1. **Qdrant (Vector Store):** Semantic search via BGE embeddings (768-dim)
2. **Elasticsearch:** BM25 lexical search with academic field mappings

### Hybrid Search with Reciprocal Rank Fusion
```python
# Retrieval flow:
Query → Query Processing → Parallel Search (BM25 + Semantic) →
Reciprocal Rank Fusion → Cohere Reranking →
Claude Generation → Citation-backed Answer
```

### RAG Pipeline Flow
```
Query → Retrieval (Hybrid Search) → Reranking (Cohere) →
Context Formation → Prompt Construction →
LLM Generation (Claude/OpenAI) → Response with Citations
```

### API Structure
- `/api/v1/documents/` - Document ingestion endpoints
- `/api/v1/search/` - Search endpoints (lexical, semantic, hybrid)
- `/api/v1/query/` - RAG query endpoints
- `/api/v1/system/` - Health checks, metrics

## Academic Paper Considerations

### Document Characteristics
- **Format:** Primarily PDFs with LaTeX formatting
- **Structure:** Abstract, Introduction, Methods, Results, Discussion, References
- **Content:** Mathematical equations, figures, tables, citations
- **Length:** 8-40 pages typically
- **Metadata:** Authors, affiliations, publication date, journal/conference, DOI, arXiv ID

### Metadata Extraction Requirements
Extract and validate:
- Title, authors, affiliations
- Publication year, venue (journal/conference)
- DOI, arXiv ID
- Abstract and keywords
- Section structure
- Citation count (when available)

### Search Query Types
The system handles four primary query types:
1. **Methodological:** "What methods were used for X?"
2. **Results-based:** "What were the findings on Y?"
3. **Comparative:** "Compare approach A vs B"
4. **Definitional:** "What is the definition of Z?"

### Citation Requirements
- All generated answers MUST include paper citations
- Citations should include: Author(s), Year, Title, Venue

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sourangshupal/Hybrid-Search-RAG](https://github.com/sourangshupal/Hybrid-Search-RAG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
