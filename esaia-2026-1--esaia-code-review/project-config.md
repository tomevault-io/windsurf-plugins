---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**ESAIA Code Review** — An AI-assisted code review CLI tool that reviews code changes against *project-specific* context (ADRs, bug history, conventions), not just universal patterns. This differentiates it from tools like CodeRabbit, which score poorly on completeness and depth for project-specific conventions.

The architecture is locked. The full system specification lives in `docs/architecture.md`; every architectural decision has a numbered ADR in `docs/adrs/`.

## Canonical Sources

- **`docs/architecture.md`** — The composed system view: 3 flows (indexing, review, evaluation), 18 modules, error policy summary, module index linking each module to its governing ADR
- **`docs/adrs/ADR-001` through `ADR-015`** — Atomic architectural decisions with full reasoning (decision drivers, options analyzed, consequences, review triggers)
- **`pesquisa-de-mercado.md`** — Market research, differentiation strategy, success metrics, product features
- **`tech_lead_guide.md`** — Planning context and sprint backlog

When working in this codebase, prefer reading the relevant ADR over `tech_lead_guide.md` for any decision-related question. ADRs are authoritative; the tech lead guide is historical context.

## Architecture (Summary)

Three flows share a single ChromaDB store with two collections (`adrs`, `bug_patterns`):

| Flow | Triggered by | Purpose |
|---|---|---|
| **Indexing** | `reviewbot init` or doc changes | Populate the context store |
| **Review** | `reviewbot review` | Review a diff against retrieved project context |
| **Evaluation** | `reviewbot eval` | Measure system quality against research metrics |

Read `docs/architecture.md` for diagrams, module narratives, and the full module index.

## Module Index (18 modules)

| Module | Responsibility | Governing ADR(s) | Flow |
|---|---|---|---|
| `src/cli.py` | Typer CLI entrypoint — `init`, `review`, `eval` | ADR-006 | All |
| `src/reviewer.py` | Orchestrates the review pipeline; owns error policy | ADR-013 | Review |
| `src/loader.py` | Read source documents, parse frontmatter | ADR-002, ADR-008 | Indexing |
| `src/chunker.py` | Split ADRs by section, bugs by field; preserve `parent_id` | ADR-009 | Indexing |
| `src/embedder.py` | Generate embeddings via `text-embedding-3-small` | ADR-003 | Indexing + Review |
| `src/context_store.py` | `VectorStore` interface + ChromaDB; two collections | ADR-002, ADR-008 | Indexing + Review |
| `src/diff_parser.py` | `unidiff` integration | ADR-005 | Review |
| `src/diff_splitter.py` | Split diffs >180K tokens by file | ADR-014 | Review |
| `src/retriever.py` | Parallel queries against both collections | ADR-002, ADR-008 | Review |
| `src/reranker.py` | Threshold + dedup; swappable `RerankStrategy` interface | ADR-010 | Review |
| `src/parent_fetcher.py` | Resolve chunk IDs to full parent documents | ADR-009 | Review |
| `src/prompt_builder.py` | Assemble prompt; mark cacheable blocks | ADR-011 | Review |
| `src/llm_client.py` | Sonnet primary, Gemini fallback, caching | ADR-001, ADR-011, ADR-013 | Review |
| `src/validator.py` | Verify cited ADR-NNN and INC-NNN exist | ADR-007 | Review |
| `src/formatter.py` | Render output; `mode={review,questions}` seam | ADR-015 | Review |
| `src/tracker.py` | Log tokens, cost, latency, cache hits, fallback rate | ADR-001, ADR-011, ADR-013 | All |
| `eval/eval_runner.py` | Run real `reviewer.py` against the corpus | ADR-012 | Eval |
| `eval/scorer.py` | Programmatic + Haiku-judge + human spot-check | ADR-012 | Eval |

## Tech Stack Decisions (Locked)

Don't propose alternatives without strong justification — each item has a numbered ADR with full reasoning:

- **Language:** Python (ADR-004)
- **CLI:** Typer (ADR-006)
- **Diff parsing:** `unidiff` library (ADR-005)
- **Vector DB:** ChromaDB embedded, two collections behind `VectorStore` interface (ADR-002, ADR-008)
- **Embeddings:** `text-embedding-3-small` from OpenAI, configurable parameter (ADR-003)
- **LLM (reviewer):** Claude Sonnet 4.6 primary, Gemini 3.1 Pro fallback (ADR-001)
- **LLM (judge / re-rank):** Haiku 4.5 (ADR-012 for judging; ADR-010 for re-ranking upgrade path)
- **Retrieval pattern:** Parent-document — chunk by section, fetch full parent for prompt (ADR-009)
- **Re-ranking (MVP):** Threshold + dedup heuristic; cross-encoder/Haiku upgrade seam (ADR-010)
- **Prompt caching:** Anthropic `cache_control` markers in `prompt_builder.py` (ADR-011)
- **Anti-hallucination:** `validator.py` programmatic citation verification (ADR-007)
- **Output mode seam:** `formatter.format(output, mode={review,questions})` from day one (ADR-015)
- **Large diff handling:** `diff_splitter.py` splits by file when prompt >180K tokens (ADR-014)

## Development Commands

Once the project is scaffolded (these don't exist yet):

```bash
# Install dependencies
pip install -e ".[dev]"

# Run the CLI
reviewbot init         # Index project context into ChromaDB
reviewbot review       # Review staged changes against indexed context
reviewbot eval         # Run evaluation harness against corpus

# Run tests
pytest tests/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ESAIA-2026-1/esaia-code-review](https://github.com/ESAIA-2026-1/esaia-code-review) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
