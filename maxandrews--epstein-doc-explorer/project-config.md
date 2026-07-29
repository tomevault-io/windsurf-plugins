---
trigger: always_on
description: An intelligent document analysis and network visualization system that processes legal documents to extract relationships, entities, and events, then visualizes them as an interactive knowledge graph.
---

# Epstein Document Network Explorer

An intelligent document analysis and network visualization system that processes legal documents to extract relationships, entities, and events, then visualizes them as an interactive knowledge graph.

## Project Overview

This project analyzes the Epstein document corpus to extract structured information about actors, actions, locations, and relationships. It uses Claude AI for intelligent extraction and presents findings through an interactive network visualization interface.

**Live Demo:** [Deployed on Render](https://epstein-doc-explorer-1.onrender.com/)

---

## Architecture Overview

The project has two main phases:

### 1. Analysis Pipeline
**Purpose:** Extract structured data from raw documents using AI
**Technology:** TypeScript, Claude AI (Anthropic), SQLite
**Location:** Root directory + `analysis_pipeline/`

### 2. Visualization Interface
**Purpose:** Interactive exploration of the extracted relationship network
**Technology:** React, TypeScript, Vite, D3.js/Force-Graph
**Location:** `network-ui/`

---

## Key Features

### Analysis Pipeline Features
- **AI-Powered Extraction:** Uses Claude to extract entities, relationships, and events from documents
- **Semantic Tagging:** Automatically tags triples with contextual metadata (legal, financial, travel, etc.)
- **Tag Clustering:** Groups 1400+ tags into 21 semantic clusters for better filtering
- **Entity Deduplication:** Merges duplicate entities using LLM-based similarity detection
- **Incremental Processing:** Supports analyzing new documents without reprocessing everything
- **Top-3 Cluster Assignment:** Each relationship is assigned to its 3 most relevant tag clusters

### Visualization Features
- **Interactive Network Graph:** Force-directed graph with 15,000+ relationships
- **Actor-Centric Views:** Click any actor to see their specific relationships
- **Smart Filtering:** Filter by 21 content categories (Legal, Financial, Travel, etc.)
- **Timeline View:** Chronological relationship browser with document links
- **Document Viewer:** Full-text document display with highlighting
- **Responsive Design:** Works on desktop and mobile devices
- **Performance Optimized:** Uses materialized database columns for fast filtering

---

## Project Structure

```
docnetwork/
├── analysis_pipeline/          # Document analysis scripts
│   ├── extract_data.py        # Initial document extraction
│   ├── analyze_documents.ts   # Main AI analysis pipeline
│   ├── cluster_tags.ts        # Tag clustering with AI
│   ├── dedupe_with_llm.ts     # Entity deduplication
│   └── extracted/             # Raw extracted documents
│
├── network-ui/                 # React visualization app
│   ├── src/
│   │   ├── components/        # React components
│   │   ├── api.ts            # Backend API client
│   │   └── App.tsx           # Main application
│   └── dist/                  # Production build
│
├── api_server.ts              # Express API server
├── document_analysis.db       # SQLite database (68MB)
├── tag_clusters.json          # 21 semantic tag clusters
├── add_top_clusters_column.ts # Migration: materialize top clusters
└── add_misc_cluster.ts        # Migration: add Misc category
```

---

## Core Components

### Analysis Pipeline

#### 1. Document Extraction (`analysis_pipeline/extract_data.py`)
**Purpose:** Extract raw text from PDF documents
**Input:** PDF files in `data/documents/`
**Output:** JSON files in `analysis_pipeline/extracted/`
**Key Features:**
- Preserves document metadata (ID, category, date)
- Handles various PDF formats
- Stores full text for AI analysis

#### 2. Document Analysis (`analysis_pipeline/analyze_documents.ts`)
**Purpose:** Main AI-powered extraction pipeline
**Input:** Extracted JSON documents
**Output:** SQLite database with entities and relationships
**Key Features:**
- Uses Claude to extract RDF-style triples (subject-action-object)
- Extracts temporal information (dates, timestamps)
- Tags relationships with contextual metadata
- Handles batch processing with rate limiting
- Stores document full text for search

**Database Schema:**
```sql
-- Documents table
CREATE TABLE documents (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  doc_id TEXT UNIQUE NOT NULL,
  file_path TEXT NOT NULL,
  one_sentence_summary TEXT NOT NULL,      -- AI-generated brief summary
  paragraph_summary TEXT NOT NULL,         -- AI-generated detailed summary
  date_range_earliest TEXT,                -- Earliest date mentioned in document
  date_range_latest TEXT,                  -- Latest date mentioned in document
  category TEXT NOT NULL,                  -- Document category
  content_tags TEXT NOT NULL,              -- JSON array of content tags
  analysis_timestamp TEXT NOT NULL,        -- When analysis was performed
  input_tokens INTEGER,                    -- Claude API usage metrics
  output_tokens INTEGER,
  cache_read_tokens INTEGER,
  cost_usd REAL,                          -- Estimated API cost
  error TEXT,                             -- Error message if analysis failed
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maxandrews/Epstein-doc-explorer](https://github.com/maxandrews/Epstein-doc-explorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
