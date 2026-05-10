---
trigger: always_on
description: Large-scale analysis and audit of public documents from Burnsville, MN's CivicWeb document site. Extract, store, and analyze textual content from various document formats using LLM-powered insights.
---

# Burnsville MN CivicWeb Document Analysis Project

## Overview
Large-scale analysis and audit of public documents from Burnsville, MN's CivicWeb document site. Extract, store, and analyze textual content from various document formats using LLM-powered insights.

**Source**: `https://burnsville.civicweb.net/document/{id}/`
**Document Types**: HTML, PDF, Images
**Document IDs**: Pre-collected list (provided by user)

## Architecture: Dockerized RAG Pipeline

### System Design
```
┌─────────────────────────────────────────────────────────────┐
│                     Docker Compose Stack                     │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Postgres   │  │   ChromaDB   │  │  Processing  │      │
│  │   Database   │  │    Vector    │  │   Worker     │      │
│  │              │  │    Store     │  │              │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                               │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              Analysis API / Interface                 │   │
│  │          (FastAPI + Claude RAG System)                │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                               │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
                    ┌──────────────┐
                    │  Claude API  │
                    │   (External) │
                    └──────────────┘
```

### Components

1. **PostgreSQL** - Primary data store
   - Document metadata
   - Extracted text
   - Analysis results
   - Better than SQLite for this scale and concurrent access

2. **ChromaDB** - Vector database
   - Document embeddings
   - Semantic search
   - RAG context retrieval

3. **Processing Worker** - Python service
   - Document download
   - Text extraction (PDF, HTML, images)
   - Tesseract OCR included in container
   - Embedding generation
   - Queue-based processing

4. **Analysis API** - FastAPI application
   - RESTful API for queries
   - Interactive chat interface
   - Batch analysis endpoints
   - Web UI for exploration

## Why This Approach

### Advantages
- **Reproducible**: Docker ensures consistent environment
- **Scalable**: Can process documents in parallel
- **Production-Ready**: Easy to deploy, monitor, and maintain
- **Isolated**: OCR and dependencies containerized
- **Queryable**: SQL + vector search capabilities
- **API-First**: Can build multiple frontends (CLI, web, notebooks)

### Tech Stack

```yaml
Services:
  postgres:14-alpine      # Lightweight, fast, reliable
  chromadb/chroma:latest  # Official ChromaDB image
  python:3.11-slim        # Processing worker + API

Python Libraries:
  # Document Processing
  - pymupdf               # PDF extraction
  - pytesseract           # OCR wrapper
  - beautifulsoup4        # HTML parsing
  - html2text             # HTML to text
  - pillow                # Image processing
  - pdf2image             # PDF to images for OCR

  # Storage
  - psycopg2-binary       # PostgreSQL driver
  - chromadb              # Vector DB client
  - sqlalchemy            # ORM

  # API & Processing
  - fastapi               # API framework
  - uvicorn               # ASGI server
  - celery                # Task queue (optional)
  - httpx                 # Async HTTP

  # LLM Integration
  - anthropic             # Claude API
  - tiktoken              # Token counting

  # Utilities
  - python-magic          # File type detection
  - tqdm                  # Progress tracking
  - pydantic              # Data validation
```

## Data Pipeline

### Stage 1: Document Download
```
Input: document_ids.txt (list of IDs)

Process:
  1. Read document IDs from file
  2. For each ID, fetch from https://burnsville.civicweb.net/document/{id}/
  3. Detect content type (HTML/PDF/Image)
  4. Save to /data/raw/{id}/
  5. Insert metadata into PostgreSQL

Database: documents table
  - id, url, content_type, file_path,
    download_timestamp, status, error_message
```

### Stage 2: Text Extraction
```
Process by type:

HTML:
  - Parse with BeautifulSoup
  - Extract main content
  - Remove scripts, styles, navigation
  - Convert to clean markdown

PDF:
  - Attempt direct text extraction with PyMuPDF
  - Calculate text density per page
  - If density < threshold, convert to images
  - Run Tesseract OCR on images
  - Combine text with page numbers

Images:
  - Preprocess (deskew, contrast)
  - Run Tesseract OCR
  - Extract text with confidence scores
  - Store low-confidence pages for review

Output: text_content in PostgreSQL + /data/text/{id}.txt
```

### Stage 3: Chunking & Embedding
```
Process:
  1. Split text into semantic chunks (~1000 tokens)
  2. Preserve document structure (pages, sections)
  3. Add overlap between chunks (200 tokens)
  4. Generate embeddings:
     - Option A: Local model (sentence-transformers)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AnthonyHerman/burnsville-mn-analysis](https://github.com/AnthonyHerman/burnsville-mn-analysis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
