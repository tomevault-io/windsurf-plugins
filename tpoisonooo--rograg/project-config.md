---
trigger: always_on
description: ROGRAG (Robustly Optimized GraphRAG) is a sophisticated GraphRAG-based system that enhances LLM performance on specialized topics through a two-stage retrieval mechanism. Also known as HuixiangDou2, this project achieves a 15% score boost on SeedBench benchmark and outperforms mainstream RAG methods.
---

# ROGRAG (HuixiangDou2) - AI Coding Agent Guide

## Project Overview

ROGRAG (Robustly Optimized GraphRAG) is a sophisticated GraphRAG-based system that enhances LLM performance on specialized topics through a two-stage retrieval mechanism. Also known as HuixiangDou2, this project achieves a 15% score boost on SeedBench benchmark and outperforms mainstream RAG methods.

**Key Highlights:**
- Two-stage retrieval for robustness (dual-level and logic form methods)
- Incremental database construction
- Enhanced fuzzy matching and structured reasoning
- Graph-based knowledge retrieval with dense retrieval support
- Multi-modal support (text, visual, and multimodal capabilities)

## Technology Stack

**Core Framework:** Python 3.8+ with asyncio support
**Key Dependencies:**
- **ML/AI**: PyTorch (≥2.0.0), Transformers (≥4.38), Sentence Transformers, BCEmbedding
- **Vector Search**: FAISS-GPU, Scikit-learn
- **Graph Database**: Neo4j with TuGraph support
- **Web Framework**: FastAPI, Uvicorn, Gradio (≥4.41)
- **Data Processing**: Pandas, NumPy (<2.0.0), OpenPyXL
- **Document Processing**: PyMuPDF, python-docx, BeautifulSoup4, readability-lxml
- **LLM Integration**: OpenAI (≥1.0.0), BCEmbedding, TikToken
- **Caching/Queue**: Redis
- **Utilities**: Loguru, Tenacity, NetworkX (≥3.0), jieba

## Architecture & Code Organization

### Directory Structure
```
huixiangdou/                    # Main package
├── main.py                     # CLI entry point with interactive mode
├── server.py                   # FastAPI HTTP server with streaming support
├── gradio_ui.py               # Gradio web interface
├── client.py                  # Client library for API testing
├── frontend/                  # Platform integrations
│   ├── lark.py               # Lark/Feishu integration
│   ├── wechat.py             # WeChat integration
│   └── lark_group.py         # Lark group chat support
├── pipeline/                  # Core processing pipelines
│   ├── parallel.py           # Parallel processing pipeline (main implementation)
│   ├── serial.py             # Serial processing pipeline
│   ├── store.py              # Knowledge storage management
│   ├── session.py            # Session management
│   └── fasta.py              # FASTA sequence processing
├── primitive/                 # Low-level utilities
│   ├── llm.py                # LLM provider implementations
│   ├── embedder.py           # Embedding implementations
│   ├── chunk.py              # Text chunking utilities
│   ├── faiss.py              # FAISS operations (17K+ lines)
│   ├── knowledge.py          # Knowledge graph operations
│   ├── reranker.py           # Reranking implementations
│   └── file_operation.py     # File handling utilities
└── service/                   # Business logic layer
    ├── retriever/            # Retrieval implementations
    │   ├── base.py          # Base retrieval interface
    │   ├── bm25.py          # BM25 retrieval
    │   ├── dense.py         # Dense vector retrieval
    │   ├── knowledge.py     # Graph-based retrieval (22K+ lines)
    │   ├── inverted.py      # Inverted index retrieval
    │   ├── web.py           # Web search integration
    │   └── logic/           # Logical reasoning retrieval with node execution
    ├── config.py            # Configuration management
    ├── graph_store.py       # Graph database operations
    ├── nlu.py               # Natural language understanding
    └── helper.py            # Utility functions
```

### Key Configuration Files

**config.ini** (TOML format):
- `[base]`: Working directory configuration
- `[store]`: Embedding and reranker model paths, API settings
- `[tugraph]`: Graph database connection settings
- `[web_search]`: Web search engine configuration (Serper)
- `[llm]`: LLM provider configurations (Alibaba Cloud, SiliconCloud, Local, Kimi, OpenAI)
- `[frontend]`: Platform-specific settings (Lark, WeChat)

**Package Configuration:**
- `setup.py`: Standard Python package setup with setuptools
  - Package name: `huixiangdou`
  - Entry point: `huixiangdou=huixiangdou.main:run`
  - Supports Python 3.8-3.11
  - Includes package data (main.py, config files, test questions)
- `requirements.txt`: Main dependencies
- `version.py`: Version management (current: 20250101)

## Build & Development Commands

### Installation
```bash
# Standard installation
pip install -r requirements.txt
pip install -e .

# Using uv (recommended for faster installs)
uv venv --python 3.13 --index https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple
source .venv/bin/activate  # Linux/macOS
# or .venv\Scripts\activate  # Windows
uv pip install -e . --index https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple
```

### Running the Application
```bash
# CLI mode
huixiangdou --work_dir workdir --config_path config.ini

# API server mode (from server.py)
python -m huixiangdou.server

# Gradio UI mode
python -m huixiangdou.gradio_ui
```

### Docker Deployment
```bash
# Use the provided run script

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tpoisonooo/ROGRAG](https://github.com/tpoisonooo/ROGRAG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
