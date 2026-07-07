---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Retrieval-Augmented Generation system over a chemistry-of-coatings corpus (436 sources: full textbooks, vendor data sheets, papers, standards — bilingual ES/EN), built **from scratch with no RAG framework** (no LangChain/LlamaIndex) so every stage — chunking, embedding, indexing, retrieval, generation — is explicit and inspectable. Runs 100% locally (Ollama + FAISS + sentence-transformers) on Apple Silicon. Every design decision, including dead ends, is documented in `docs/00` through `docs/07` (in Spanish) — read the relevant one before changing that stage's behavior; this file only summarizes.

## Commands

```bash
# Setup (macOS)
brew install poppler tesseract ollama
ollama pull qwen2.5:7b-instruct
brew services start ollama
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

# Full pipeline, in order (each stage reads the previous stage's output)
python -m src.extraction.run        # data/raw/        -> data/processed/*/pages.jsonl
python -m src.chunking.run          # pages.jsonl       -> chunks.jsonl
python -m src.embeddings.run        # chunks.jsonl      -> embeddings.npy
python -m src.indexing.build_index  # embeddings.npy    -> vectorstore/index.faiss + metadata.jsonl
python -m src.evaluation.run_eval   # runs the 15-question test set, writes eval/results/<timestamp>.json

# Tests
pytest                              # whole suite
pytest tests/test_chunker.py        # one module
pytest tests/test_chunker.py::test_name -v   # one test

# Interactive
streamlit run app.py                          # chat UI
jupyter notebook notebooks/pipeline_completo.ipynb   # narrated walkthrough with real outputs
```

There is no lint/format command configured in this repo.

## Pipeline architecture

Each stage is a `src/<stage>/` package with a `run.py` (or `build_index.py`) CLI entrypoint and reads/writes flat files under `data/processed/<book-slug>/` — there is no database. Stages must be re-run in order after any upstream change (e.g. re-chunking requires re-embedding, since `embeddings.npy` row *i* corresponds positionally to `chunks.jsonl` line *i*, with no other key tying them together).

```
data/raw/ (PDFs, not committed)
  -> extraction    (src/extraction) -> data/processed/<slug>/pages.jsonl
  -> chunking      (src/chunking)   -> chunks.jsonl   (token-based, chapter-bounded)
  -> embeddings    (src/embeddings) -> embeddings.npy (row i <-> chunks.jsonl line i)
  -> indexing      (src/indexing)   -> vectorstore/index.faiss + metadata.jsonl (not committed)
  -> retrieval     (src/retrieval)  -> FAISS top-20 -> cross-encoder rerank -> top-5
  -> generation    (src/generation) -> grounded, cited answer via Ollama
  -> evaluation    (src/evaluation) -> deterministic metrics, no LLM-as-judge
```

`src/generation/answer.py::answer_question()` is the single entrypoint that ties retrieval + generation together — the notebook, `app.py`, and `run_eval.py` all call this same function, not the individual stages.

### Extraction (`src/extraction/`)

- Two source layouts, distinguished by **directory name**, not content sniffing: a folder named `[Author_Year]_Title` is a whole book (each PDF inside = one chapter); anything else is a category, and every loose PDF inside becomes its own standalone cited document. This distinction can't be inferred from file contents — both layouts look like "a flat folder of PDFs" on disk.
- Chapter numbers come from filename conventions accumulated empirically from real files (see the doc comment at the top of `chapters.py` for the exact patterns) — when a new book doesn't chapter-detect correctly, add a pattern there rather than special-casing the book.
- `pdf_text.py` auto-falls-back to Tesseract OCR per-page when PyMuPDF's native text extraction returns near-nothing (scanned pages) — this is a per-page decision, not a per-book one.
- Single-PDF books derive chapters from the embedded PDF table of contents (`chapters.py::chapter_map_from_toc`); multi-PDF books derive them from filenames.

### Chunking (`src/chunking/`)

- Token-based (`tiktoken`, `cl100k_base`), 600 tokens with 100 overlap, and **never crosses a chapter boundary** — chunking is done independently per `(book, chapter)` group. If you change chunk size/overlap, re-run embeddings and indexing after.

### Embeddings & retrieval (`src/embeddings/`, `src/retrieval/`, `src/indexing/`)

- Embedding model `intfloat/multilingual-e5-base` requires the `"query: "` / `"passage: "` prefix convention (see `embedder.py`) — omitting it degrades retrieval quality; it's not just a formatting nicety.
- Retrieval is two-stage: FAISS `IndexFlatIP` (bi-encoder, fetches top-20 broadly) -> `BAAI/bge-reranker-base` cross-encoder (precise, rescoring only those 20 down to top-5). Don't skip the reranker or shrink `fetch_k` too close to `top_k` — the entire point is giving the reranker a wide net.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Halexoh/RAG-chemistry](https://github.com/Halexoh/RAG-chemistry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
