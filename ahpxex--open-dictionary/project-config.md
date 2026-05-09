---
trigger: always_on
description: This repository is not a generic "dictionary scripts" project.
---

# Open Dictionary Rewrite Charter

This repository is not a generic "dictionary scripts" project.
It is the rewrite line for a reproducible dictionary production system built
on top of Wiktionary / Wiktextract data.

The system must be designed as a staged data pipeline with explicit schemas,
explicit run metadata, and deterministic handoff points between stages.

## Product Framing

- Source of truth: Wiktionary / Wiktextract data, not Wikidata.
- Canonical unit: one headword equals one entry.
- Entry model: word-centric, not "word plus part-of-speech" as the top-level unit.
- LLM goal: produce structured, Chinese learner-friendly dictionary entries from
  curated source data.
- Human editorial authority: when the pipeline hits ambiguous curation questions,
  the user decides the editorial rule. The agent must not invent permanent
  curation policy without explicit approval.

## Core Workflow

The intended end-state workflow is:

1. Download a Wiktionary / Wiktextract snapshot.
2. Ingest the raw snapshot into PostgreSQL.
3. Build curated tables from raw tables.
4. Run LLM enrichment on curated entries.
5. Export stable distributable artifacts such as JSONL and SQLite.

Expressed as data layers:

- `raw`: source-faithful imported data.
- `curated`: normalized, cleaned, word-centric entries.
- `llm`: structured generated outputs plus generation metadata.
- `exports`: packaged read-only outputs for downstream distribution.
- `meta`: run tracking, versions, prompts, and operational audit data.

No stage may skip over the previous stage's contract.
For example, LLM enrichment must consume curated entries, not raw imported blobs.

## Technical Framework

### 1. Raw Ingestion Layer

Responsibilities:

- Download source snapshots.
- Record source identity, origin URL, timestamps, and hashes.
- Load raw payloads into PostgreSQL with minimal semantic mutation.
- Preserve source payloads well enough to re-run downstream stages.

Requirements:

- Raw ingestion must be idempotent at the run level.
- Every ingestion run must have a `run_id`.
- Every raw row must be traceable to a source snapshot and ingestion run.
- The code must not mix download logic, parsing logic, and curation logic in the
  same stage module.

### 2. Curation Layer

Responsibilities:

- Read from raw PostgreSQL tables only.
- Normalize Wiktionary's fragmented record shape into a canonical word-centric
  entry model.
- Remove technical noise and transform source structure into a stable contract
  for downstream enrichment.
- Produce new tables rather than mutating raw tables in place.

Requirements:

- Curation rules must be explicit and reviewable.
- Editorial ambiguity must be escalated to the user.
- The curation layer must record its input run, rule version, and output run.
- Curation must be restartable and safe to re-run.

Examples of questions that require user decisions:

- Which tags, qualifiers, and metadata are useful vs noise.
- How multiple parts of speech should be represented under one headword.
- Which examples, derivations, pronunciations, etymologies, and related forms
  are mandatory, optional, or discardable.
- How aggressively duplicate or overlapping senses should be merged.

### 3. LLM Enrichment Layer

Responsibilities:

- Consume curated entries only.
- Build prompt inputs from stable curated payloads.
- Generate structured outputs under strict schema validation.
- Persist generation metadata and outputs in PostgreSQL.

Requirements:

- Prompt versions must be explicit and tracked.
- Model name, provider, temperature, and other generation settings must be
  recorded per run.
- Input hash, output payload, failure reason, retry count, and timestamps must
  be recorded.
- Concurrency, retry, rate limiting, and resume behavior are required
  infrastructure, not optional helpers.
- A run must be resumable without duplicating successful rows.

The LLM layer must be auditable. "We called the model and saved the result" is
not enough.

### 4. Export Layer

Responsibilities:

- Produce stable distribution artifacts from curated and/or enriched tables.
- Export to JSONL, SQLite, and other read-only formats as needed.
- Treat exported artifacts as build outputs, not mutable working data.

Requirements:

- Exports must record the exact upstream run ids they were built from.
- Export scripts must never contain business logic that belongs in curation or
  LLM enrichment.

## Mandatory Reproducibility Rules

- Every stage must have a durable `run_id`.
- Every stage must record its upstream dependency run ids.
- Prompt text must be versioned.
- Table schemas must be migrated explicitly, never created implicitly by random
  business logic at runtime.
- Runtime code must not quietly `ALTER TABLE` as a substitute for migrations.
- Raw, curated, and LLM tables must be separable by stage and by run lineage.
- Outputs must be derivable from stored source data and stored configuration.

If a feature makes the pipeline less reproducible, less inspectable, or less
restartable, it is the wrong design.

## Repository Architecture

The rewrite should converge toward a structure similar to:

- `src/open_dictionary/cli.py`
  Entry point only. Parse commands and dispatch to stage runners.
- `src/open_dictionary/config/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ahpxex/open-dictionary](https://github.com/ahpxex/open-dictionary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
