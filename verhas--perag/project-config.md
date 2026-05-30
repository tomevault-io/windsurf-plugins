---
trigger: always_on
description: provides a local, private RAG (Retrieval-Augmented Generation) pipeline that enriches
---

# Perago — Personal RAG Toolkit

## Project Overview

**Perago** (Latin: *to carry through to completion*) is a personal productivity tool for
non-developers who work with textual documents (PDF, Word, Markdown, plain text). It
provides a local, private RAG (Retrieval-Augmented Generation) pipeline that enriches
prompts with relevant context retrieved from a personal document collection.

The command-line tool is called `perag`. It is intentionally not a service — no server,
no daemon, no cloud dependency. It runs locally, stores data locally, and is invoked
from the command line or by Claude Code via a SKILL.md.

---

## Repository Structure

```
perago/
├── CLAUDE.md                  # This file
├── README.md                  # User-facing documentation
├── pyproject.toml             # Root package definition (if monorepo build)
├── config.example.toml        # Example configuration file
│
├── perag/                     # Main CLI package (entry point: perag <subcommand>)
│   ├── __init__.py
│   ├── cli.py                 # Subcommand dispatcher (chunk/embed/ingest/query)
│   ├── config.py              # Config loading from config.toml
│   └── schema.py              # Shared JSON chunk schema (dataclass/TypedDict)
│
├── chunkers/                  # Format-aware chunkers (one module per format)
│   ├── __init__.py
│   ├── base.py                # Abstract base class: Chunker.chunk(path) -> [Chunk]
│   ├── pdf.py                 # PDF chunking via pdfplumber
│   ├── docx.py                # Word chunking via python-docx
│   ├── markdown.py            # Markdown chunking via markdown-it-py
│   ├── text.py                # Plain text / paragraph-aware fallback
│   └── registry.py            # Maps file extension -> Chunker class
│
├── embedders/                 # Embedding providers (one module per provider)
│   ├── __init__.py
│   ├── base.py                # Abstract base class: Embedder.embed([str]) -> [[float]]
│   ├── ollama.py              # Ollama HTTP API
│   ├── openai.py              # OpenAI embeddings API
│   ├── local.py               # sentence-transformers (fully local, no API key)
│   └── registry.py            # Maps provider name -> Embedder class
│
├── db/                        # sqlite-vec database layer
│   ├── __init__.py
│   ├── store.py               # Schema init, upsert, meta table management
│   └── search.py              # ANN query, returns top-k chunks
│
├── tests/
│   ├── fixtures/              # Sample PDF, DOCX, MD, TXT files for testing
│   ├── test_chunkers.py
│   ├── test_embedders.py
│   ├── test_db.py
│   └── test_pipeline.py       # End-to-end: chunk -> embed -> ingest -> query
│
├── skills/
│   └── SKILL.md               # Claude Code skill: how to use perag from Claude Code
│
└── docs/
    ├── chunking.md            # How chunking works per format
    ├── embedders.md           # Supported embedding providers and configuration
    └── pipeline.md            # Full pipeline walkthrough
```

---

## Subcommand Design

All subcommands read/write JSON on stdin/stdout, making the pipeline composable:

```bash
# Full pipeline (piped)
perag chunk document.pdf | perag embed | perag ingest

# Full pipeline (with intermediate files for inspection/debugging)
perag chunk document.pdf      > chunks.json
perag embed   < chunks.json   > chunks_embedded.json
perag ingest  < chunks_embedded.json

# Query
perag query "what are the termination conditions?"
```

### `perag chunk <file>`
- Detects format from extension
- Dispatches to the appropriate chunker in `chunkers/`
- Outputs a JSON array of Chunk objects to stdout

### `perag embed`
- Reads JSON array of Chunk objects from stdin
- Calls the configured embedding provider in batches
- Outputs the same JSON array with `vector` field added

### `perag ingest`
- Reads JSON array of embedded Chunk objects from stdin
- Writes to the sqlite-vec database
- Enforces dimension and model name consistency via the `meta` table
- Upserts by `id` (re-ingesting an updated document replaces existing chunks)

### `perag init`
- Creates `.perag/` in the current directory
- Writes a minimal `config.toml` inheriting from `~/.perag/config.toml` if it exists
- Adds `.perag/perag.db` to `.gitignore` if a `.gitignore` is present
- Safe to re-run — never overwrites an existing config

### `perag query "<text>"`
- Embeds the query text using the configured provider
- Performs ANN search against the sqlite-vec database
- Outputs top-k chunks as plain text (suitable for Claude Code context injection)
- `--json` flag outputs structured JSON instead

---

## JSON Chunk Schema

Every chunk flowing through the pipeline conforms to this schema:

```json
{
  "id":                 "contracts/nda_2024.pdf::chunk::7",
  "source":             "contracts/nda_2024.pdf",
  "content":            "The agreement shall terminate upon 30 days written notice...",
  "metadata": {
    "format":           "pdf",
    "page":             3,
    "section":          "Termination"
  },
  "embedding_model":    "nomic-embed-text",
  "embedding_provider": "ollama",
  "vector":             [0.021, -0.134, 0.087, "..."]
}
```

After `perag chunk`, the fields `embedding_model`, `embedding_provider`, and `vector`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [verhas/perag](https://github.com/verhas/perag) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-30 -->
