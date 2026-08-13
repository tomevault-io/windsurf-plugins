---
trigger: always_on
description: A personal knowledge engine that transforms passive reading into structured understanding of the AI field. Ingests sources (papers, articles, videos, podcasts), extracts claims with evidence, discovers connections, generates ideas through multi-agent debate, and maintains a living model of the current state of AI — tracking capabilities, limitations, bottlenecks, breakthroughs, and anticipations across thematic areas.
---

# Knowledge Base — Agent Instructions

A personal knowledge engine that transforms passive reading into structured understanding of the AI field. Ingests sources (papers, articles, videos, podcasts), extracts claims with evidence, discovers connections, generates ideas through multi-agent debate, and maintains a living model of the current state of AI — tracking capabilities, limitations, bottlenecks, breakthroughs, and anticipations across thematic areas.

## Architecture

Local-first. PostgreSQL + pgvector + FTS as the single database. Markdown memory files (human-auditable). SQLite job queue. Telegram interface via long polling. File-based library (`library/{source_id}/`). Gateway → Adapters (Telegram, CLI, Heartbeat) → Skills Registry → Agent Executor → Storage. No cloud dependencies.

## The Three Layers

- **Knowledge graph** — what sources say. Claims, concepts, edges, evidence. Every claim links back to a verbatim snippet.
- **Landscape model** — what is true about AI right now. Themes (DAG), capabilities (with maturity), limitations (with type and trajectory), bottlenecks (with resolution horizon), breakthroughs (with implication cascades), anticipations (trackable predictions), cross-theme implications.
- **Belief system** — what you think about what is true. Tracked positions with confidence, evidence for/against, landscape links, history. The system surfaces conflicts and challenges stale positions.

## Core Principles

- **Evidence-traced.** Every claim links back to a verbatim snippet. Every edge has evidence from both sides.
- **Temporally aware.** The landscape model is a trajectory, not a snapshot. State summaries describe evolution, momentum, and stalling.
- **Human-enriched.** Automated extraction detects what's explicit and implicit. Highest-value insights come from the user via `/enrich`, `/implications`, and `/challenge`.
- **Limitations are the most valuable signal.** The system extracts implicit limitations alongside explicit ones, classifies them by type and severity, and links them to bottlenecks.

## Generated Wiki Indexes

Do not hand-edit generated wiki index rows in `wiki/index.md` or `wiki/*/index.md`. Index display rows are derived artifacts built from canonical page frontmatter/body. If an index row is wrong, fix the underlying page or metadata, then rebuild the index with `retrieval.wiki_index.rebuild_index()` or the relevant narrower rebuild helper. If an index shows mojibake, regenerate it from clean pages; do not copy terminal output back into Markdown.

## How to Invoke Skills

All skills can be invoked via the test harness:

```bash
python scripts/test_skill_harness.py <skill_name> "<slash_command>"
```

**Requirements:** PostgreSQL must be running with the knowledge base schema. Environment variables in `.env` must be configured (see `.env.example`).

**What to expect:** The harness imports the handler, constructs a job object, and executes it. Output is printed to stdout. Some skills stream progress updates. Long-running skills (save, reflect deep, synthesis) may take several minutes.

Alternatively, read the skill's prompt file at `skills/prompts/<skill_name>.md` and follow the instructions directly using your own tools to query the database.

---

## Skills Reference

### Ingestion & Management

#### save
Ingest a URL into the library with full extraction pipeline (theme classification, claim extraction, landscape signals, cross-theme implications, anticipation matching, belief checking).

**Usage:** `/save <url> [time_ranges]`
**Examples:**
- `/save https://arxiv.org/abs/2401.12345`
- `/save https://www.youtube.com/watch?v=abc123 8:55-27:04, 2:34:58-2:46:17`

**Invoke:** `python scripts/test_skill_harness.py save "/save <url>"`
**Handler:** `gateway/save_handler.py` (direct)

---

#### summarise
Preview a source summary before deciding to fully ingest. Fetches URL, classifies themes, generates summary. Does NOT run claim extraction or landscape pipeline. Stages result for 24h; follow up with `/save_confirmed` to complete ingestion.

**Usage:** `/summarise <url> [time_ranges]`
**Examples:**
- `/summarise https://arxiv.org/abs/2401.12345`
- `/summarise https://www.youtube.com/watch?v=abc123 8:55-27:04`

**Invoke:** `python scripts/test_skill_harness.py summarise "/summarise <url>"`
**Handler:** `gateway/summarise_handler.py` (direct)

---

#### delete
Remove a source and all derived artifacts. Cascading DB cleanup with no LLM involvement — removes claims, concepts, edges, themes, capabilities, limitations, bottlenecks, implications, anticipations, breakthroughs, beliefs evidence, and the `library/{source_id}/` directory.

**Usage:** `/delete <source_id_or_url>`
**Examples:**
- `/delete 01HQ3XMVBN7K9P2R4S6T8W0Y1Z`
- `/delete https://arxiv.org/abs/2401.12345`

**Invoke:** `python scripts/test_skill_harness.py delete "/delete <source_id>"`
**Handler:** `gateway/delete_handler.py` (direct)

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sehyungp92/knowledge_base](https://github.com/sehyungp92/knowledge_base) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
