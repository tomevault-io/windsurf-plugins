---
trigger: always_on
description: This toolkit extracts knowledge from arbitrary source materials (PDFs, articles, books, web content) and synthesizes it into a structured, searchable knowledge base of **concepts** and **theses**. The result is a machine-readable knowledge layer that enables LLM agents to perform deep, grounded, multi-source analysis on a domain.
---

# Knowledge Base Builder — Agent Instructions

## Purpose

This toolkit extracts knowledge from arbitrary source materials (PDFs, articles, books, web content) and synthesizes it into a structured, searchable knowledge base of **concepts** and **theses**. The result is a machine-readable knowledge layer that enables LLM agents to perform deep, grounded, multi-source analysis on a domain.

The methodology was proven on ~10,000 pages of scanned financial reference materials and ~100 web articles, producing 358 concept documents and 48 thesis documents with hybrid BM25 + semantic search. This generalized version captures the complete methodology for any domain.

---

## Quick Start

If you're a fresh agent and the knowledge base already has content:

1. Load `extracted/concepts/PRIMER.md` — orients your analytical framework
2. Search via `python3 search.py "your query" --top 10 -v --related`
3. Read 5-15 relevant concept docs, follow cross-references
4. Read thesis docs for analytical depth
5. Go to source docs (`extracted/sources/`) for raw detail

If you're building the knowledge base from scratch, read the rest of this file.

---

## Architecture Overview

For full design rationale, see **SYSTEM.md**. Summary:

```
{project}/
├── CLAUDE.md                          # This file — methodology & status
├── SYSTEM.md                          # Architecture & design rationale
├── search.py                          # Thin shim → kb_engine (keeps `python3 search.py`)
├── build_embeddings.py                # Thin shim → kb_engine (dense build, provider-driven)
├── engine/kb_engine/                  # Shared retrieval engine (installable as kb-engine)
├── input_docs/                        # Place raw source materials here (PDFs, articles, etc.)
├── extracted/
│   ├── concepts/
│   │   ├── PRIMER.md                  # Agent orientation document
│   │   ├── docs/                      # Concept synthesis documents
│   │   └── theses/                    # Cross-cutting thesis documents
│   └── sources/                       # One subdirectory per source
│       ├── {source-1-slug}/           # Raw extractions from source 1
│       └── {source-2-slug}/           # Raw extractions from source 2
└── templates/
    ├── concept.md                     # Template for concept documents
    └── thesis.md                      # Template for thesis documents
```

Three layers, each serving a different retrieval purpose:

- **Layer 1: Concepts** (~1,000 words each) — "what is X?" and "how does X work?"
- **Layer 2: Theses** (1,500-3,000 words each) — analytical essays threading multiple concepts together
- **Layer 3: Source extractions** — full raw material for deep dives

---

## Source Canon

Document every source here. For each source, record:

### Source Template

```markdown
### Source N: {Name}

**Status:** {Not started | Extraction in progress | Fully integrated}

**Location:** {URL, file path, or description of where to find the material}

**Topic coverage:** {What domains/subjects does this source cover?}

**Bias/perspective:** {What worldview or analytical framework does this source use? What's it strong on? What's it weak on? What should be cross-referenced?}

**Format:** {Scanned PDF / text PDF / web articles / book / video transcripts / etc.}

**Extraction methodology:** {Which approach — see Extraction Methodology below}

**Integration status:**
- Extracted: {N} files in `extracted/sources/{slug}/`
- Concepts generated: {N} in `extracted/concepts/docs/`
- Theses generated: {N} in `extracted/concepts/theses/`
```

---

## Extraction Methodology

### Source Materials

Place all raw source documents (PDFs, saved articles, etc.) in the `input_docs/` directory. This is the canonical location for source materials before extraction. The extraction process reads from `input_docs/` and writes structured output to `extracted/sources/{source-slug}/`.

### Strategy Selection

Choose your approach based on source format:

| Source Format | Primary Approach | Fallback |
|---------------|-----------------|----------|
| **Scanned PDF** (no text layer) | Claude vision (chunk-per-agent) | Tesseract OCR |
| **Text PDF** (has text layer) | `pdftotext` or direct Read | Claude vision for charts |
| **Web articles** | WebFetch/WebSearch + manual chart annotation | Browser automation (Chrome) |
| **Text files / markdown** | Direct Read | — |
| **Books (digital)** | Chapter-by-chapter vision reads | pdftotext + chart annotation |

### Approach 1: Claude Vision (chunk-per-agent)

**Best for:** Scanned PDFs, image-heavy documents, anything with charts/diagrams.

**Method:** Split large PDFs into 50-page chunks, dispatch one subagent per chunk for vision-based transcription.

```
For each 50-page chunk:
1. Agent reads the PDF pages as images via the Read tool with `pages` parameter
2. Agent transcribes all text verbatim
3. Agent describes all charts/graphs using structured format (see below)
4. Agent writes output as markdown with page references

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [j-wang/knowledge-agent-template](https://github.com/j-wang/knowledge-agent-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
