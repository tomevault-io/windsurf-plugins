---
trigger: always_on
description: A comprehensive Python system for discovering, scraping, filtering, and enriching Australian cyber security events from multiple data sources using machine learning and LLM-based analysis. The pipeline combines Perplexity AI search, OAIC breach notifications, Google Custom Search, Webber Insurance data, and optional GDELT feeds to create a structured, deduplicated database of cyber incidents with detailed metadata, affected entities, and ASD risk classifications.
---

# Australian Cyber Events Discovery and Enrichment Pipeline

A comprehensive Python system for discovering, scraping, filtering, and enriching Australian cyber security events from multiple data sources using machine learning and LLM-based analysis. The pipeline combines Perplexity AI search, OAIC breach notifications, Google Custom Search, Webber Insurance data, and optional GDELT feeds to create a structured, deduplicated database of cyber incidents with detailed metadata, affected entities, and ASD risk classifications.

## Tech Stack

| Layer | Technology | Version | Purpose |
|-------|------------|---------|---------|
| Runtime | Python | 3.8+ | Core scripting and data processing |
| Framework | Pydantic | 2.x | Data validation and models |
| LLM Primary | OpenAI GPT-4o | Latest | Primary event extraction and enrichment |
| LLM Fallback | GPT-4o-mini | Latest | Fast initial filtering during discovery |
| AI Search | Perplexity AI | API v1 | Real-time cyber event discovery and fact-checking |
| Database | SQLite | 3.x | Event storage and persistence |
| Web Scraping | Playwright | 1.44+ | Browser-based content extraction |
| Data Processing | Pandas | 2.x | Data manipulation and analysis |
| ML Classification | scikit-learn | 1.3+ | Random Forest event filtering |
| PDF Processing | pdfplumber, tabula-py | Latest | OAIC PDF and Power BI dashboard extraction |
| Cloud APIs | Google BigQuery | 3.14+ | GDELT data source (optional, expensive) |
| CLI | Typer | 0.15+ | Command-line interface |

## Quick Start

### Prerequisites

- Python 3.8+
- API keys for:
  - OpenAI (OPENAI_API_KEY) - required for LLM enrichment
  - Perplexity AI (PERPLEXITY_API_KEY) - recommended for event discovery
  - Google Custom Search (optional) - for Google Search data source
  - Google Cloud BigQuery (optional, expensive) - only for GDELT source

### Installation

```bash
# Clone the repository
git clone https://github.com/colinpriest/australian-cyber-events-scraper.git
cd australian-cyber-events-scraper

# Install dependencies
pip install -r requirements.txt

# For OAIC Power BI dashboard scraping (optional):
playwright install chromium

# Create and configure environment variables
cp .env.example .env
# Edit .env with your API keys
```

### Development

```bash
# Run the unified pipeline (discovers, enriches, deduplicates, classifies, generates dashboard)
python run_full_pipeline.py

# Quick rolling refresh (last 90 days, recommended for monthly updates)
python pipeline.py refresh

# Check pipeline status (last ingest + latest event)
python pipeline.py status

# Run tests
pytest

# Generate dashboard only (if data already exists)
python run_full_pipeline.py --dashboard-only
```

### Available Commands

| Command | Description |
|---------|-------------|
| `python pipeline.py refresh` | Rolling 3-month refresh with recommended sources |
| `python pipeline.py status` | Report last ingest and latest event |
| `python run_full_pipeline.py` | Full pipeline with all 5 phases |
| `python run_full_pipeline.py --discover-only` | Discovery phase only (auto-enriches with Perplexity) |
| `python run_full_pipeline.py --dashboard-only` | Generate dashboard from existing data |
| `python scripts/asd_risk_classifier.py` | Classify events with ASD risk matrix |
| `python scripts/build_static_dashboard.py` | Generate interactive HTML dashboard |
| `pytest` | Run test suite |

## Project Structure

```
australian-cyber-events-scraper/
├── pipeline.py                    # Simplified CLI entry point (refresh/status/rebuild)
├── run_full_pipeline.py           # Advanced 5-phase pipeline entry point
├── requirements.txt
│
├── cyber_data_collector/          # Core package
│   ├── datasources/               # Perplexity, OAIC, Google Search, Webber, GDELT
│   ├── enrichment/                # High-quality 5-stage enrichment pipeline
│   │   ├── high_quality_enrichment_pipeline.py
│   │   ├── content_acquisition.py
│   │   ├── gpt4o_enricher.py
│   │   ├── perplexity_fact_checker.py
│   │   ├── enrichment_validator.py
│   │   └── enrichment_audit_storage.py
│   ├── filtering/
│   │   ├── rf_event_filter.py     # Random Forest ML filter
│   │   ├── progressive_filter.py
│   │   └── confidence_filter.py
│   ├── models/                    # CyberEvent, EventSeverity, CyberEventType
│   ├── pipelines/
│   │   └── discovery.py           # Discovery and initial processing pipeline
│   ├── processing/                # LLM classification, deduplication, enrichment
│   ├── storage/
│   │   ├── cyber_event_data_v2.py # Thread-safe SQLite database operations
│   │   ├── database.py
│   │   └── deduplication_storage.py
│   ├── utils/
│   │   ├── entity_scraper.py      # Playwright-based web scraping
│   │   ├── llm_extractor.py       # GPT-4o-mini event extraction
│   │   └── ...
│   └── tests/
│

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [colinpriest/australian-cyber-events-scraper](https://github.com/colinpriest/australian-cyber-events-scraper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
