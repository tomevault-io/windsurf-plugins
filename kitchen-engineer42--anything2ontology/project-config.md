---
trigger: always_on
description: Anything2Ontology is a knowledge management and modelling pipeline that converts various media formats into a comprehensive ontology for coding agents. The pipeline transforms inputs (files, URLs, repos) into structured knowledge that can be used by AI coding assistants to build applications.
---

# Anything2Ontology - Project Context

## Overview
Anything2Ontology is a knowledge management and modelling pipeline that converts various media formats into a comprehensive ontology for coding agents. The pipeline transforms inputs (files, URLs, repos) into structured knowledge that can be used by AI coding assistants to build applications.

## Key Design Principles

### 1. Agile Schema Design
Schemas have two parts:
- **Fixed part**: Standard fields that are always present
- **JIT (Just-In-Time) part**: Flexible metadata that agents can define case-by-case

Example: `ParseResult` has fixed fields (source_path, status) and a JIT `metadata` dict.

### 2. Loose Coupling
Modules are independent and communicate through well-defined interfaces (schemas). Each module can be developed, tested, and modified independently.

### 3. Load Context As Needed
Like Claude Code's SKILL.md approach - read headers first to decide whether to load full content. Minimize context loading until necessary.

### 4. Atomic Tools
Human behavioral sequences expressed in natural language should be encapsulated into atomic, deterministic tools (parsers, extractors).

### 5. Dual-Format Logging
All operations generate both:
- JSON logs (for machine parsing)
- Plain text logs (for human reading)

## Project Structure

```
Anything2Ontology/
├── src/
│   ├── anything2markdown/     # Module 1: Universal parser
│   │   ├── parsers/           # File parsers (MarkItDown, MinerU, PaddleOCR-VL, Tabular)
│   │   ├── url_parsers/       # URL parsers (YouTube, Bilibili, FireCrawl, Repomix)
│   │   ├── utils/             # Logging, file utils, retry logic
│   │   ├── schemas/           # ParseResult schema
│   │   ├── router.py          # Routing logic
│   │   ├── pipeline.py        # Main orchestration
│   │   └── cli.py             # CLI interface (anything2md)
│   ├── markdown2chunks/       # Module 2: Smart chunking
│   │   ├── chunkers/          # HeaderChunker, LLMChunker
│   │   ├── utils/             # Token estimation, Levenshtein
│   │   ├── schemas/           # Chunk, ChunksIndex
│   │   ├── router.py          # Markdown vs JSON routing
│   │   ├── pipeline.py        # Main orchestration
│   │   └── cli.py             # CLI interface (md2chunks)
│   ├── chunks2skus/           # Module 3: Knowledge extraction
│   │   ├── extractors/        # Factual, Relational, Procedural, Meta
│   │   ├── utils/             # Logging, LLM client
│   │   ├── schemas/           # SKU, LabelTree, Glossary, Index
│   │   ├── router.py          # Load chunks, route to extractors
│   │   ├── pipeline.py        # Main orchestration
│   │   └── cli.py             # CLI interface (chunks2skus)
│   └── skus2ontology/         # Module 4: Ontology assembly
│       ├── utils/             # Logging, LLM client (with multi-turn)
│       ├── schemas/           # OntologyManifest, ChatSession
│       ├── assembler.py       # Copy SKUs, rewrite paths
│       ├── chatbot.py         # Interactive spec.md generation
│       ├── readme_generator.py # Template-based README.md
│       ├── pipeline.py        # Main orchestration
│       └── cli.py             # CLI interface (skus2ontology)
├── input/                     # User input files
├── output/                    # Module 1 output (flat structure)
│   ├── chunks/                # Module 2 output (chunked markdown)
│   ├── passthrough/           # JSON files (unchanged)
│   └── skus/                  # Module 3 output (knowledge units)
├── ontology/                  # Module 4 output (self-contained ontology)
├── logs/                      # JSON and text logs
└── module_design/             # Design docs for each module
```

## Module 1: Anything2Markdown

### Purpose
Convert various file types and URLs into Markdown or JSON for downstream processing.

### Routing Logic
| Input | Parser |
|-------|--------|
| PDF (normal) | MarkItDown |
| PDF (scanned/low quality) | PaddleOCR-VL (fallback) |
| PPT, DOC, media | MarkItDown |
| xlsx, csv | TabularParser (→ JSON) |
| YouTube URL | YouTubeParser |
| Bilibili URL | BilibiliParser (subtitles or faster-whisper) |
| GitHub repo | RepomixParser |
| Other URLs | FireCrawlParser |

### CLI Commands
```bash
anything2md init          # Create directories
anything2md run           # Run full pipeline
anything2md run -v        # Verbose mode
anything2md parse-file X  # Parse single file
anything2md parse-url X   # Parse single URL
```

## Language Configuration

Set `LANGUAGE` in `.env` to control the output language of all LLM prompts:
- `LANGUAGE=en` (default) — English prompts, English output
- `LANGUAGE=zh` — Chinese prompts, Chinese output (titles, descriptions, definitions, README, mapping, eureka)

All prompt constants are `dict[str, str]` keyed by language code. At each call site, the prompt is selected via `PROMPT[settings.language]`. JSON keys, field names, enum values, and format specifications remain English in both languages.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kitchen-engineer42/Anything2Ontology](https://github.com/kitchen-engineer42/Anything2Ontology) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
