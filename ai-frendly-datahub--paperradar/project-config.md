---
trigger: always_on
description: PaperRadar is an academic paper and research-signal collection platform. It aggregates arXiv RSS/API, lab blogs, citation feeds, and benchmark/implementation sources, extracts entities (authors, venues, techniques), stores them in DuckDB, and generates HTML reports.
---

# PaperRadar - Project Architecture

## Overview

PaperRadar is an academic paper and research-signal collection platform. It aggregates arXiv RSS/API, lab blogs, citation feeds, and benchmark/implementation sources, extracts entities (authors, venues, techniques), stores them in DuckDB, and generates HTML reports.

## Architecture

### Standard Tier Pattern

PaperRadar follows the Standard Tier template with single `paperradar/` package:

```
PaperRadar/
├── main.py                      # CLI: python main.py --category research
├── paperradar/                  # Single package (not radar/)
│   ├── collector.py             # collect_sources() - RSS + paper APIs
│   ├── analyzer.py              # apply_entity_rules() - keyword matching
│   ├── reporter.py              # generate_report() - Jinja2 HTML
│   ├── storage.py               # RadarStorage - DuckDB upsert/query
│   ├── models.py                # Paper, Source, EntityDefinition
│   ├── config_loader.py         # YAML loading
│   ├── search_index.py          # SQLite FTS5 search
│   ├── raw_logger.py            # JSONL raw data logging
│   ├── nl_query.py              # Natural language query parsing
│   ├── logger.py                # structlog configuration
│   ├── common/                  # Shared utilities
│   └── mcp_server/              # MCP server (future)
├── config/
│   ├── config.yaml              # database_path, report_dir, etc.
│   └── categories/
│       └── research.yaml        # AI/ML sources + entities
├── data/                        # DuckDB, search_index.db, raw/ JSONL
├── reports/                     # Generated HTML reports
├── tests/
│   ├── unit/                    # Unit tests (80%+ coverage)
│   └── integration/             # Integration tests
└── .github/workflows/           # GitHub Actions
```

## Core Components

### 1. Collector (paperradar/collector.py)

Implements RSS plus multiple paper-source collectors:

| Collector | API | Rate Limit | Auth | Parse |
|-----------|-----|-----------|------|-------|
| RSSCollector | RSS/Atom feeds | feed dependent | None | title, summary, published |
| ArxivCollector | arxiv.org/api/query | 3 req/sec | None | title, authors, abstract, arxiv_id, pdf_url |
| SemanticScholarCollector | api.semanticscholar.org | 100 req/5min | Optional | title, authors, venue, citations |
| PubMedCollector | eutils.ncbi.nlm.nih.gov | Unlimited | None | title, authors, journal, PMID |
| BiorxivCollector | api.biorxiv.org | Free | None | title, authors, DOI, abstract |
| OpenAlexCollector | api.openalex.org | 10 req/sec | None | title, authors, concepts, citations |
| CrossRefCollector | api.crossref.org | Free | None | DOI, title, authors, journal |

**Key patterns:**
- `@retry` decorator with exponential backoff
- Timeout enforcement (default 15s)
- Error handling per source (doesn't fail entire pipeline)
- Returns `tuple[list[Paper], list[str]]` (papers, errors)

### 2. Analyzer (paperradar/analyzer.py)

Entity extraction via keyword matching:

```python
def apply_entity_rules(papers: list[Paper], entities: list[EntityDefinition]) -> list[Paper]:
    # Matches keywords in title + abstract + authors
    # Populates paper.matched_entities dict
```

**Entities in research.yaml:**
- Research Areas: ML, NLP, CV, RL, etc.
- Key Researchers: Hinton, LeCun, Bengio, etc.
- Institutions: Stanford, MIT, Google Brain, OpenAI, etc.
- Venues: NeurIPS, ICML, ICLR, ACL, CVPR, etc.
- Techniques: Transformer, BERT, GPT, Diffusion, etc.

### 3. Storage (paperradar/storage.py)

DuckDB-based paper storage:

```sql
CREATE TABLE papers (
  paper_id VARCHAR PRIMARY KEY,
  title VARCHAR NOT NULL,
  authors VARCHAR[],
  abstract VARCHAR,
  url VARCHAR,
  pdf_url VARCHAR,
  arxiv_id VARCHAR,
  doi VARCHAR,
  venue VARCHAR,
  publication_date DATE,
  citation_count INTEGER,
  categories VARCHAR[],
  keywords VARCHAR[],
  collected_at TIMESTAMP,
  source_name VARCHAR,
  category VARCHAR
);
```

**Methods:**
- `upsert_papers(papers)` - Insert/update by paper_id
- `recent_papers(category, days)` - Query recent papers
- `delete_older_than(days)` - Retention policy

### 4. Reporter (paperradar/reporter.py)

Jinja2 HTML report generation with:
- Academic theme (gradient header, paper cards)
- Stats dashboard (sources, collected, matched, window)
- Paper cards with title, authors, abstract preview, venue, citations
- Error reporting
- Responsive grid layout
- Dark mode ready

### 5. Search Index (paperradar/search_index.py)

SQLite FTS5 full-text search:

```python
with SearchIndex(db_path) as idx:
    idx.upsert(paper_id, title, abstract, authors)
    results = idx.search("transformer BERT")
```

### 6. Models (paperradar/models.py)

Core dataclasses:

```python
@dataclass
class Paper:
    title: str
    link: str
    abstract: str
    authors: list[str]
    published: datetime | None
    source: str
    category: str
    doi: str | None = None
    arxiv_id: str | None = None
    pdf_url: str | None = None
    venue: str | None = None
    citation_count: int | None = None
    categories: list[str] = field(default_factory=list)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ai-frendly-datahub/PaperRadar](https://github.com/ai-frendly-datahub/PaperRadar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
