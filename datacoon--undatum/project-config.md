---
trigger: always_on
description: <!-- OPENSPEC:START -->
---

<!-- OPENSPEC:START -->
# OpenSpec Instructions

These instructions are for AI assistants working in this project.

Always open `@/openspec/AGENTS.md` when the request:
- Mentions planning or proposals (words like proposal, spec, change, plan)
- Introduces new capabilities, breaking changes, architecture shifts, or big performance/security work
- Sounds ambiguous and you need the authoritative spec before coding

Use `@/openspec/AGENTS.md` to learn:
- How to create and apply change proposals
- Spec format and conventions
- Project structure and guidelines

Keep this managed block so 'openspec update' can refresh the instructions.

<!-- OPENSPEC:END -->

# undatum — Agent Guide

## Project Overview

**undatum** is a Python command-line tool for data processing and analysis. It provides a unified interface for converting, analyzing, validating, and transforming data across multiple formats with a focus on low memory footprint through streaming.

- **Repository**: https://github.com/datacoon/undatum
- **Version**: 1.6.0
- **License**: MIT
- **Author**: Ivan Begtin <ivan@begtin.tech>
- **Python requirement**: >= 3.9

### Key Capabilities

- Multi-format I/O: CSV, JSON Lines, BSON, XML, XLS, XLSX, Parquet, AVRO, ORC, YAML, TSV, plus 140+ formats via `iterabledata` (geospatial, lakehouse, scientific; see `docs/FORMAT_SUPPORT.md`)
- Compression: ZIP, XZ, GZ, BZ2, ZSTD, LZ4, 7Z (codec profiles `fast`/`balanced`/`max` via iterabledata)
- Streaming processing for large files via `iterabledata`
- Automatic encoding, delimiter, and file type detection
- Data validation with built-in and custom rules
- Statistics and field analysis (including DuckDB-accelerated stats)
- Filtering and querying with expressions
- Schema generation and Frictionless Data Packaging
- Database ingestion: MongoDB, PostgreSQL, DuckDB, MySQL, SQLite, Elasticsearch
- AI-powered dataset documentation via multiple LLM providers
- Optional read-only Data API over files (FastAPI + DuckDB)
- Extensible plugin system

## Technology Stack

| Layer | Libraries |
|-------|-----------|
| CLI framework | `typer`, `click` |
| Console output | `rich` |
| Data processing | `pandas`, `duckdb`, `iterabledata` |
| Serialization | `orjson`, `jsonlines`, `bson`, `xmltodict`, `pyyaml`, `avro`, `pyorc` |
| Compression | `lz4`, `py7zr`, `pyzstd` |
| Excel | `openpyxl`, `xlrd`, `xlwt` |
| Validation | `validators`, `pydantic` |
| Query/filter | `mistql` |
| AI providers | `requests` (OpenAI, OpenRouter, Ollama, LM Studio, Perplexity) |
| Testing | `pytest`, `pytest-cov`, `pytest-benchmark` |
| Linting/formatting | `black`, `ruff`, `pylint`, `mypy` |

## Project Structure

```
undatum/
├── __init__.py           # Package metadata (__version__, __author__)
├── __main__.py           # CLI entry point (undatum / data commands)
├── core.py               # Main Typer app and command registration
├── constants.py          # File type lists, date patterns, EU themes
├── utils.py              # General utilities
├── ai/                   # AI provider implementations
│   ├── base.py           # AIService abstract base and errors
│   ├── config.py         # AI configuration loading
│   ├── perplexity.py     # Legacy Perplexity provider
│   ├── providers.py      # OpenAI, OpenRouter, Ollama, LM Studio providers
│   └── schemas.py        # Pydantic schemas for AI output
├── cmds/                 # Individual CLI command implementations (~40 modules)
│   ├── analyzer.py       # analyze command
│   ├── api.py            # api subcommands
│   ├── converter.py      # convert command
│   ├── db_load.py        # db load command
│   ├── db_query.py       # db query command
│   ├── doc.py            # doc command (dataset documentation)
│   ├── extractor.py      # extract command (PDF/table extraction)
│   ├── pipeline.py       # pipeline command
│   ├── statistics.py     # stats command
│   ├── validator.py      # validate command
│   └── ...               # Many more commands
├── common/               # Shared utilities used by commands
│   ├── chunked_io.py     # Chunked file I/O helpers
│   ├── db_connection.py  # Database connection management
│   ├── duckdb_config.py  # DuckDB configuration helpers
│   ├── engine_selector.py# Processing engine selection
│   ├── errors.py         # UndatumError hierarchy and error handling
│   ├── filter.py         # Data filtering logic
│   ├── functions.py      # Dict helper functions (dot-notation access)
│   ├── iterable.py       # Iterable data wrappers
│   ├── masking.py        # Data masking utilities
│   ├── parallel.py       # Parallel processing helpers
│   ├── path_utils.py     # Path and S3 URI utilities
│   ├── pipeline_parser.py# Pipeline DSL parser
│   ├── progress.py       # Progress bar helpers
│   ├── s3_iterable.py    # S3-backed iterable data
│   ├── schema_utils.py   # Schema generation helpers
│   ├── scheme.py         # URL scheme handlers
│   └── validation_rules.py# Built-in validation rules
├── formats/              # Format-specific handlers
│   ├── docx.py           # Word document I/O
│   └── s3.py             # S3 writer and client
├── plugins/              # Plugin system
│   ├── base.py           # Plugin base classes
│   ├── manager.py        # Plugin loading and registration
│   └── registry.py       # Plugin registry

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [datacoon/undatum](https://github.com/datacoon/undatum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
