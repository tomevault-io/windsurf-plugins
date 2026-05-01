---
trigger: always_on
description: Insurance RAG — a multi-domain Retrieval-Augmented Generation system for US insurance industries. It uses a domain-plugin architecture where each insurance line (Medicare, Auto, Property, etc.) registers its own data sources, extractors, enrichment rules, topic definitions, query patterns, and system prompt. Shared infrastructure handles embeddings, vector storage, chunking, hybrid retrieval, and LLM generation.
---

# AGENTS.md

## Project Overview

Insurance RAG — a multi-domain Retrieval-Augmented Generation system for US insurance industries. It uses a domain-plugin architecture where each insurance line (Medicare, Auto, Property, etc.) registers its own data sources, extractors, enrichment rules, topic definitions, query patterns, and system prompt. Shared infrastructure handles embeddings, vector storage, chunking, hybrid retrieval, and LLM generation.

Currently implemented domains: **Medicare** (fully functional) and **Auto Insurance** (scaffolded).

**Language:** Python 3.11+
**Package manager:** pip / setuptools (see `pyproject.toml`)

## Repository Layout

```
src/insurance_rag/           # Main package (installed as editable via `pip install -e .`)
  __init__.py
  config.py                 # Paths, env config, multi-domain support (ACTIVE_DOMAINS, domain_data_dir, etc.)
  domains/                  # Domain plugin system
    __init__.py              #   Domain registry (register_domain, get_domain, list_domains)
    base.py                  #   InsuranceDomain abstract base class
    medicare/                #   Medicare domain plugin
      __init__.py             #   MedicareDomain class (source kinds: iom, mcd, codes)
      patterns.py             #   LCD patterns, query expansion, synonyms, system prompt
      data/topics.json        #   Medicare clinical/policy topic definitions
    auto/                    #   Auto Insurance domain plugin
      __init__.py             #   AutoInsuranceDomain class (source kinds: regulations, forms, claims, rates)
      patterns.py             #   Coverage patterns, query expansion, synonyms, system prompt
      states.py               #   State-specific config (tort system, min liability, PIP, top markets)
      data/topics.json        #   Auto insurance topic definitions
  download/                 # Phase 1: download data (Medicare-specific downloaders)
    __init__.py              #   Re-exports download_iom, download_mcd, download_codes
    iom.py                   #   IOM chapter PDF scraper
    mcd.py                   #   MCD bulk ZIP downloader
    codes.py                 #   HCPCS + ICD-10-CM code file downloader
    _manifest.py             #   Manifest writing and SHA-256 hashing
    _utils.py                #   URL sanitization, stream_download helper
  ingest/                   # Phase 2: text extraction, enrichment, chunking, clustering, summarization
    __init__.py              #   SourceKind type
    extract.py               #   PDF/text extraction (pdfplumber, optional unstructured fallback)
    enrich.py                #   HCPCS/ICD-10 semantic enrichment
    chunk.py                 #   LangChain text splitters; optional summary generation
    cluster.py               #   Topic clustering (loads topic defs from active domain)
    summarize.py             #   Extractive summarization (TF-IDF sentence scoring)
  index/                    # Phase 3: embedding and vector store
    __init__.py              #   Re-exports get_embeddings, get_or_create_chroma, upsert_documents
    embed.py                 #   sentence-transformers embeddings
    store.py                 #   ChromaDB upsert (collection_name from domain); get_raw_collection helper
  query/                    # Phase 4: retrieval and RAG chain
    __init__.py
    retriever.py             #   Domain-aware retriever with query expansion and topic-summary boosting
    expand.py                #   Cross-source query expansion (patterns loaded from active domain)
    hybrid.py                #   HybridRetriever: semantic + BM25 via RRF with cross-source diversification
    chain.py                 #   Local LLM RAG chain (system prompt from domain)
app.py                      # Streamlit UI with domain selector (launch: `streamlit run app.py`)
scripts/                    # CLI entry points (all support --domain flag)
  download_all.py            #   Bulk download (--domain medicare|auto|all, --source, --force)
  ingest_all.py              #   Extract, chunk, embed, store (--domain, --source, --force, --skip-*)
  validate_and_eval.py       #   Index validation + retrieval eval (hit rate, MRR)
  query.py                   #   Interactive RAG REPL (--domain, --filter-source, --filter-state)
  run_rag_eval.py            #   Full-RAG eval report generation
  eval_questions.json        #   Eval question set (expected keywords/sources)
tests/                      # Unit tests (pytest; install with pip install -e ".[dev]")
  conftest.py                #   Shared fixtures (autouse: reset BM25 index after each test)
  test_domains.py            #   Domain registry, interface compliance, domain-specific tests
  test_config.py             #   Safe env int/float parsing
  test_download.py           #   Mocked HTTP, idempotency, zip-slip and URL sanitization
  test_ingest.py             #   Extraction and chunking tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [csmangum/insurance_rag](https://github.com/csmangum/insurance_rag) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-27 -->
