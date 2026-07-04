---
trigger: always_on
description: This document describes the agent responsibilities and workflows for the Hungarian Health Portal web scraping project. It defines how different components work together to achieve RAG-optimized data extraction.
---

# AGENTS.md

## Hungarian Health Portal Web Scraper - Agent Architecture

This document describes the agent responsibilities and workflows for the Hungarian Health Portal web scraping project. It defines how different components work together to achieve RAG-optimized data extraction.

---

## 1. Overview

The scraper uses a **multi-agent architecture** where specialized components handle different aspects of the scraping and processing pipeline:

```
┌─────────────────────────────────────────────────────────────────┐
│                    HealthScraper (Orchestrator)                  │
│  • Coordinates all agents                                        │
│  • Manages configuration and state                               │
│  • Handles output and error reporting                            │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      JinaClient (Extractor)                      │
│  • Handles Jina AI API communication                             │
│  • Rate limiting and retry logic                                 │
│  • Content extraction from web pages                             │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    ContentProcessor (Analyzer)                   │
│  • Cleans and normalizes content                                 │
│  • Semantic chunking for RAG                                     │
│  • Summary generation                                            │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   TOONConverter (Formatter)                      │
│  • Converts JSON to TOON format                                  │
│  • Token optimization for LLM processing                         │
│  • Output formatting                                             │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Agent Descriptions

### 2.1 HealthScraper (Orchestrator Agent)

**Purpose**: Main coordinator for the entire scraping pipeline.

**Responsibilities**:
- Initialize and configure all other agents
- Manage crawl state (visited URLs, queue, depth tracking)
- Coordinate the workflow between agents
- Handle errors and recovery
- Generate output reports

**Input**:
- Configuration (from `.env` or environment variables)
- Command-line arguments

**Output**:
- Scraped pages (ScrapedPage objects)
- Processed RAG documents
- Output files (JSON, TOON)
- Error reports

**Workflow**:
```python
# Initialize
config = Config.load()
scraper = HealthScraper(config)

# Crawl phase
pages = scraper.crawl(start_url, max_pages, max_depth)

# Process phase  
output_files = scraper.process_results(pages)
```

**Error Handling**:
- Graceful degradation on individual page failures
- Retry logic for transient errors
- Fallback to error page representation

---

### 2.2 JinaClient (Extractor Agent)

**Purpose**: Handle all communication with the Jina AI Reader API.

**Responsibilities**:
- API authentication and request signing
- Rate limiting to respect API constraints
- Retry logic with exponential backoff
- Error translation and handling
- Response parsing and validation

**Input**:
- URLs to extract
- Extraction options (raw, info, links)

**Output**:
- Extracted content (markdown format)
- Page metadata (title, language, content type)
- Discovered links

**Configuration**:
```python
JinaConfig(
    api_key="your-api-key",
    base_url="https://api.jina.ai",
    timeout=30,
    max_retries=3,
    retry_delay=1.0,
)
```

**Rate Limiting**:
- Default: 60 requests per minute
- Configurable via `REQUESTS_PER_MINUTE` env var

**Error Handling**:
- `JinaClientError` for API failures
- Automatic retry on 5xx errors
- Immediate failure on 401 (auth error)
- Backoff on 429 (rate limit)

---

### 2.3 ContentProcessor (Analyzer Agent)

**Purpose**: Transform raw content into RAG-optimized chunks.

**Responsibilities**:
- Content cleaning and normalization
- Semantic section extraction
- Token-aware chunking
- Summary generation
- Metadata enrichment

**Input**:
- Raw markdown content
- URL and page metadata

**Output**:
- RAGChunk objects
- RAGDocument objects
- Token counts
- Section boundaries

**Processing Pipeline**:

```
Raw Content
    │
    ▼
┌───────────────┐
│  Clean/       │ → Remove excessive whitespace
│  Normalize    │ → Fix encoding issues
└───────────────┘
    │
    ▼
┌───────────────┐
│  Extract      │ → Find markdown headings
│  Sections     │ → Identify semantic sections
└───────────────┘
    │
    ▼
┌───────────────┐
│  Generate     │ → Create 2-3 sentence summary
│  Summary      │ → Capture key information
└───────────────┘
    │
    ▼
┌───────────────┐
│  Chunk        │ → Split by token limit

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kevsuin/web-scraper](https://github.com/kevsuin/web-scraper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
