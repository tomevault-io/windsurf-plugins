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

# AGENTS.md - Metacrafter

This document provides guidance for AI agents working with the Metacrafter codebase.

## Overview

Metacrafter is a Python command-line tool and library for labeling table fields and data files. It uses rule-based classification to identify:
- Personal Identifiable Information (PII)
- Person names, surnames, midnames
- Basic identifiers (UUID/GUID, email, phone, etc.)
- Country/language-specific identifiers
- Dates and times
- Various semantic data types

## Repository Structure

```
metacrafter/
├── metacrafter/          # Main package
│   ├── __init__.py       # Package initialization, exports exceptions
│   ├── __main__.py       # CLI entry point
│   ├── core.py           # Main CLI command handler (CrafterCmd)
│   ├── config.py         # Configuration file loader (.metacrafter)
│   ├── exceptions.py     # Custom exception classes
│   ├── classify/         # Core classification engine
│   │   ├── processor.py  # RulesProcessor - loads and applies rules
│   │   ├── stats.py      # Analyzer - field statistics and analysis
│   │   └── utils.py      # Utility functions
│   ├── core/             # Core validation utilities
│   │   └── validators.py # Validation functions
│   ├── registry/         # Registry client integration
│   │   └── client.py     # Client for metacrafter-registry
│   └── server/           # API server components
│       ├── api.py        # API endpoints
│       └── manager.py    # Server management
├── rules/                # Default rule files (YAML)
│   ├── basic/           # Basic identifier rules
│   ├── common/          # Common rules (dates, internet, etc.)
│   ├── pii/             # PII detection rules
│   ├── en/              # English-specific rules
│   ├── ru/              # Russian-specific rules
│   └── fr/              # French-specific rules
├── tests/               # Test suite
├── scripts/             # Utility scripts
└── setup.py            # Package setup
```

## Key Components

### 1. Core Engine (`metacrafter/core.py`)

The `CrafterCmd` class is the main entry point for all operations:
- `scan_file()` - Scan data files (CSV, JSON, Parquet, etc.)
- `scan_db()` - Scan SQL databases
- `scan_mongodb()` - Scan MongoDB databases
- `scan_data()` - Scan in-memory data (list of dicts)
- `scan_bulk()` - Scan multiple files in a directory

### 2. Rules Processor (`metacrafter/classify/processor.py`)

`RulesProcessor` handles:
- Loading YAML rule files from configured paths
- Compiling rules (text, PyParsing, function-based)
- Applying rules to field names and data values
- Filtering by context, language, country codes
- Confidence scoring

### 3. Statistics Analyzer (`metacrafter/classify/stats.py`)

`Analyzer` computes field statistics:
- Data type detection (str, int, float, dict, etc.)
- Uniqueness metrics
- Length statistics (min, max, avg)
- Character analysis (digits, alphas, special chars)
- Dictionary value detection

### 4. Configuration (`metacrafter/config.py`)

`ConfigLoader` reads `.metacrafter` YAML config files from:
- Current working directory
- User home directory (`~/.metacrafter`)

Configuration options:
- `rulepath`: List of directories containing rule YAML files

## Rule System

Rules are YAML files that define how to identify data types. Three match types:

1. **text** - Exact text matching (for field names)
   ```yaml
   midname:
     key: person_midname
     match: text
     type: field
     rule: midname,secondname,middlename
   ```

2. **ppr** - PyParsing pattern matching (for data values)
   ```yaml
   rukadastr:
     key: rukadastr
     match: ppr
     type: data
     rule: Word(nums, min=1, max=2) + Literal(':')...
   ```

3. **func** - Python function validation
   ```yaml
   runpabyfunc:
     key: runpa
     match: func
     type: data
     rule: metacrafter.rules.ru.gov.is_ru_law
   ```

## Supported File Formats

Metacrafter uses `iterabledata` package for file format support:

**Text formats:** CSV, TSV, JSON, JSONL, XML
**Binary formats:** BSON, Parquet, Avro, ORC, Excel (XLS/XLSX), Pickle
**Compression:** gzip, bzip2, xz, lz4, zstandard, Brotli, Snappy, ZIP

Format detection is automatic based on file extension.

## Database Support

- **SQL databases:** Any database supported by SQLAlchemy (PostgreSQL, MySQL, SQLite, SQL Server, Oracle, DuckDB, etc.)
- **NoSQL:** MongoDB (via pymongo)

## Common Tasks

### Adding a New Rule

1. Create or edit a YAML file in `rules/` directory (or custom rulepath)
2. Define rule with appropriate match type (text/ppr/func)
3. Set metadata: key, name, type (field/data), priority, contexts, langs, country
4. Test with `metacrafter scan file test.csv`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apicrafter/metacrafter](https://github.com/apicrafter/metacrafter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
