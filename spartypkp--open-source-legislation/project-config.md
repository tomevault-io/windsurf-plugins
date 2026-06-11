---
trigger: always_on
description: > ⚠️ **PROJECT ARCHIVED - NO LONGER MAINTAINED**
---

# CLAUDE.md

> ⚠️ **PROJECT ARCHIVED - NO LONGER MAINTAINED**
>
> This project is no longer actively developed or supported. All SQL download links are broken,
> the infrastructure has been shut down, and scrapers may be outdated. This documentation is
> preserved as a technical reference for the architecture that was built, but should not be
> treated as guidance for an active project.
>
> See [README.md](README.md) for the full deprecation notice.

---

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Open-source-legislation is a platform for democratizing access to global legislative data. It provides scraped and processed legislation from countries and jurisdictions worldwide in a unified SQL schema format. The project enables developers to build legal applications using primary source legislation data without the typical barriers to accessing this information.

**Key Features:**
- Unified PostgreSQL schema with pgvector support for embeddings
- Hierarchical node-based legislation modeling
- Cross-corpus and cross-jurisdiction reference tracking
- Pydantic-based data models for validation and type safety
- LLM-ready with pre-generated embeddings
- 50+ US state and federal legislation corpora (various stages of completion)
- Python SDK with Instructor library integration

## Architecture

### Core Components

**Scraping Pipeline (3-Phase Architecture):**
1. **Read Phase** (`1_read.py`): Extracts top-level title links from table of contents pages
2. **Scrape Phase** (`2_scrape_regular.py`, `2a_scrape_selenium.py`): Scrapes legislative content using regular HTTP requests or Selenium for complex sites
3. **Process Phase** (`3_process.py`): Processes scraped data, generates embeddings, and establishes node relationships

**Data Models (Pydantic-based):**
- `Node`: Core legislation model with structure/content types
- `NodeID`: Hierarchical identifier system (e.g., `us/ca/statutes/title=1/chapter=2/section=3`)
- `NodeText`: Paragraph-based text content with reference tracking
- `DefinitionHub`: Legal term definitions with scope and inheritance
- `ReferenceHub`: Cross-references between legislation nodes

**Database Schema:**
- PostgreSQL with unified schema across jurisdictions
- Table naming: `{country}_{jurisdiction}_{corpus}` (e.g., `us_ca_statutes`)
- Support for graph traversal and cross-corpus connections

### Directory Structure

```
open-source-legislation/
├── src/
│   ├── scrapers/
│   │   ├── us/
│   │   │   ├── (states)/           # Organizing directory
│   │   │   │   ├── {state}/        # e.g., ca, ny, tx
│   │   │   │   │   └── statutes/
│   │   │   │   │       ├── read{STATE}.py    # Phase 1: Extract TOC links
│   │   │   │   │       ├── scrape{STATE}.py  # Phase 2: Scrape content
│   │   │   │   │       ├── process{STATE}.py # Phase 3: Generate embeddings
│   │   │   │   │       └── data/             # Scraped data (gitignored)
│   │   │   └── federal/
│   │   │       ├── usc/            # US Code
│   │   │       ├── ecfr/           # Electronic Code of Federal Regulations
│   │   │       └── aim/            # Aeronautical Information Manual
│   │   └── mhl/                    # Other countries (e.g., Marshall Islands)
│   │       └── federal/
│   ├── 1_SCRAPE_TEMPLATE/          # Template for new scrapers
│   │   ├── 1_read.py
│   │   ├── 2_scrape_regular.py
│   │   ├── 2a_scrape_selenium.py   # For JavaScript-heavy sites
│   │   └── 3_process.py
│   ├── utils/
│   │   ├── pydanticModels.py       # Core data models (Node, NodeID, NodeText)
│   │   ├── scrapingHelpers.py      # Scraping utilities & node insertion
│   │   ├── processingHelpers.py    # Embedding generation (batch processing)
│   │   ├── utilityFunctions.py     # Database, API clients, chat completion
│   │   ├── legislation_metadata.json # Jurisdiction metadata & status
│   │   └── api_pricing.json        # API cost tracking
│   └── github/
│       ├── progressTracker.py      # Auto-generate status tables
│       ├── scraper_status.json     # Current scraper status
│       └── status_table.md         # Markdown status table
├── docs/
├── deprecated/
├── public/                         # Images for README
├── requirements.txt
├── .env                           # Database & API credentials (gitignored)
├── CLAUDE.md                      # This file
├── README.md
└── contributing.md
```

**Important Notes:**
- Directories in parentheses like `(states)` are organizational only - they help group related scrapers but are filtered out when parsing paths
- Each scraper follows the naming pattern `{ACTION}{STATE_CODE}.py` (e.g., `readCA.py`, `scrapeTX.py`)
- The `data/` directory in each scraper is gitignored and stores intermediate scraping results

## Common Development Tasks

### Environment Setup

1. **Clone and Setup Virtual Environment:**
   ```bash
   git clone https://github.com/spartypkp/open-source-legislation.git
   cd open-source-legislation

   # Create virtual environment
   python3 -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate

   # Install dependencies
   pip install -r requirements.txt
   ```

2. **Set Python Path:**
   ```bash
   # Adjust path to your actual repository location

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spartypkp/open-source-legislation](https://github.com/spartypkp/open-source-legislation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
