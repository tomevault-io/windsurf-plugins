---
trigger: always_on
description: A command-line Python tool that takes two EPUB files (an original and its translation) and produces a single combined EPUB where each paragraph from the original is immediately followed by its translated counterpart. The output preserves the original book's formatting, structure, and pagination.
---

# CLAUDE.md — Bilingual EPUB Creator

## Project Overview

A command-line Python tool that takes two EPUB files (an original and its translation) and produces a single combined EPUB where each paragraph from the original is immediately followed by its translated counterpart. The output preserves the original book's formatting, structure, and pagination.

## Quick Reference

- **Language:** Python 3.10+
- **Package manager:** pip with pyproject.toml
- **Entry point:** `python -m bilingual_epub <original.epub> <translated.epub> -o output.epub`
- **Test command:** `pytest tests/`
- **Lint:** `ruff check src/`
- **Format:** `ruff format src/`

## Project Structure

```
bilingual-epub/
├── CLAUDE.md
├── pyproject.toml
├── README.md
├── src/
│   └── bilingual_epub/
│       ├── __init__.py
│       ├── cli.py                # Argument parsing, main entry point
│       ├── epub_parser.py        # EPUB unpacking, spine reading, paragraph extraction
│       ├── alignment.py          # Paragraph alignment engine (pluggable strategy)
│       ├── similarity.py         # Similarity scoring functions
│       ├── merger.py             # Combines aligned pairs into new XHTML documents
│       └── epub_writer.py        # Assembles and writes the final EPUB file
└── tests/
    ├── conftest.py               # Shared fixtures (sample EPUB builders)
    ├── test_epub_parser.py
    ├── test_alignment.py
    ├── test_similarity.py
    ├── test_merger.py
    ├── test_epub_writer.py
    └── test_integration.py       # End-to-end: two EPUBs in, one bilingual EPUB out
```

## Dependencies

- `ebooklib` — reading and writing EPUB files
- `beautifulsoup4` with `lxml` — parsing and manipulating XHTML content
- `numpy` — matrix operations for alignment dynamic programming
- `click` — CLI framework
- `pytest` — testing
- `ruff` — linting and formatting

Do not add any dependency on LLM APIs or large ML models. Alignment must work offline with classical algorithms.

## Architecture and Data Flow

### Pipeline

```
Original EPUB ─► parse ─► flat list of ParagraphNodes ─┐
                                                        ├─► align ─► merged XHTML ─► output EPUB
Translated EPUB ─► parse ─► flat list of ParagraphNodes ─┘
```

### Step 1: EPUB Parsing (`epub_parser.py`)

An EPUB is a ZIP archive. Use `ebooklib` to open it and read the OPF spine, which defines the reading order of XHTML documents. For each XHTML document in spine order, use BeautifulSoup to walk the DOM and extract block-level text elements.

Define a `ParagraphNode` dataclass:

```python
@dataclass
class ParagraphNode:
    text: str                  # Plain text content (for alignment)
    html_element: Tag          # Original BeautifulSoup Tag (preserves formatting)
    source_file: str           # Which XHTML file this came from
    index_in_file: int         # Position within that file
    global_index: int          # Position across the entire book
    element_tag: str           # e.g. "p", "h1", "blockquote"
```

Extract from these tags: `p`, `h1`–`h6`, `blockquote`, `li`, and `div` elements that directly contain text (not wrapper divs). Skip elements that contain no meaningful text (e.g., purely whitespace, or elements containing only images). Preserve image-only elements in a separate list so they can be reinserted at the correct positions in the output.

**Critical for Challenge #2:** Flatten all paragraphs from all XHTML files in spine order into a single list per book. This erases the arbitrary file-boundary differences between the two editions. Pagination boundaries are reintroduced later based on the original book's file structure.

### Step 2: Paragraph Alignment (`alignment.py`, `similarity.py`)

This is the core algorithmic challenge. The two books may have slightly different paragraph counts due to translation choices, editorial changes, or redactions. We need to find the best global alignment between the two paragraph sequences.

#### Similarity Scoring (`similarity.py`)

Implement a pluggable `SimilarityScorer` protocol:

```python
class SimilarityScorer(Protocol):
    def score(self, a: ParagraphNode, b: ParagraphNode) -> float:
        """Return similarity score in [0, 1]. Higher = more similar."""
        ...
```

**Primary implementation: `HybridSimilarityScorer`**

This combines multiple lightweight signals. Each signal produces a score in [0, 1], and the final score is a weighted sum.

1. **Length ratio (weight: 0.4):** The character counts of translated paragraph pairs are roughly proportional. Compute `score = 1 - abs(len_a * expected_ratio - len_b) / max(len_a * expected_ratio, len_b)` where `expected_ratio` is estimated from the median length ratio of all paragraphs in both books (compute this once upfront as a preprocessing step). This is inspired by the Gale-Church algorithm, which is the gold standard for bilingual sentence alignment in computational linguistics.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/auroflow) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
