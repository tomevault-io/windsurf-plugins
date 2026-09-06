---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

WikDict-gen is a Python-based dictionary generator that extracts multilingual translation data from dbnary dumps (loaded into an embedded pyoxigraph RDF store) to create various dictionary formats. It supports 26 languages and generates dictionaries in multiple formats (SQLite, TEI XML, Kobo, StarDict).

## Development Commands

### Setup
```bash
make download     # Fetch the dbnary dumps into virtuoso/ttl/
make              # Full build: load the store, generate all dictionaries, run checks
uv sync           # Install dependencies into .venv (done automatically by `uv run`)
```

### Testing
```bash
make test         # Run all unit tests
uv run -m unittest discover -s src
```

### Build Targets
```bash
uv run src/run.py load-store  # Bulk-load the dbnary dumps into the pyoxigraph store
make raw          # Build the store if needed, then query it into raw dictionaries
make processed    # Process raw dictionaries
make generic      # Create generic SQLite dictionaries
make wdweb        # Generate WikDict web format dictionaries
make check        # Check for empty databases
```

### Cleaning
```bash
make clean        # Remove dictionaries directory
make distclean    # Remove dictionaries and venv
```

### Quick Dictionary Lookup
```bash
src/run.py search de en haus  # Search German-English for "haus"
```

## Architecture

### Data Pipeline
1. **pyoxigraph store** (`src/sparql/store.py`): bulk-loads dbnary TTL dumps, each edition into its own named graph, into an embedded on-disk RDF store (`dictionaries/store`)
2. **Raw Extraction** (`src/sparql/`): SPARQL queries (`queries.py`) run against the store via `backend.py`; single-edition queries are scoped to one graph, cross-edition ones query the union. Output goes to raw SQLite databases
3. **Processing** (`src/process.py`): Processes raw data, adds language-specific processing
4. **Inference** (`src/infer.py`): Generates translation inferences between language pairs
5. **Output Generation**: Creates multiple dictionary formats

### Scoring model
All translation-scoring constants live in **`src/scoring.py`** (the single source
of truth): the direct/reverse/indirect path weights, the `AggByScore` step, and
the `is_good`/TEI thresholds. They are substituted into `src/infer.sql` by
`infer.py:render_sql()` (plain SQL can't import Python) and imported by
`generic.py`/`tei.py`. A target reached by several pivot paths gets the plain
`sum` of their scores: corroboration from many pivots is evidence *for* a
translation, so there is deliberately no decay/penalty on extra paths. Two
penalty schemes that fought this were tried and measured net-negative on the
precision gold, so neither shipped: a diminishing-returns combine and a per-path
fan-out (polysemy) penalty. The fan-out penalty did lift gold precision
(0.91→0.97) but dropped ~67% of *correct* gap-fills (retention 1.00→0.33), so
combined F0.5 fell (see architecture.md #4). Change a constant here, then re-run
the harness.

### Evaluating the scoring model
`src/evaluate.py` measures a scoring change instead of eyeballing it. It holds out
each language pair's human-curated direct translations, lets inference reproduce
them through the other languages as pivots, and reports recall at the `is_good`
threshold (plus the `gehen → gå`/`gehen → åka` probes). It reads a built
`dictionaries/infer.sqlite3` and needs no rebuild.

Held-out reconstruction measures *recall* well but counts every gap-fill (no
direct edge — the translations inference exists to produce) as a false positive,
so it is blind to precision. **`src/precision_gold.py`** supplies the missing
half: a checked-in gold of inferred-only gap-fills
(`src/tests/data/precision_gold.tsv`), labelled correct/incorrect by an
independent LLM judge and hand-spot-checked, so it is independent of the dbnary
edges we infer from. When that gold is present, `evaluate` combines the two —
held-out → recall, gold → precision — into one F0.5 objective, and also reports
how many labelled-correct gap-fills a variant *retains* (the guardrail against
dropping good translations).
```bash
uv run src/run.py evaluate              # combined held-out recall + gold precision
uv run src/run.py evaluate --sweep      # sweep the is_good threshold
uv run src/run.py evaluate --experiment # production vs fan-out (polysemy) variants
uv run src/run.py mine-gold --out cand.tsv  # (re)sample gap-fills to label for the gold
make evaluate                           # the --sweep run
```
The gold's labels are a frozen artifact (relabelling needs model access); mining
is reproducible and decoupled from it. The confirmed-bad rows are the
known-false-positive list (`precision_gold.confirmed_bad()`) the testing.md item
asked for. A scoring change ships only if it improves the combined objective and
keeps the probes green.

### Key Components

- **`src/run.py`**: Main entry point with subcommand dispatch
- **`src/helper.py`**: Utilities for language handling and database operations
- **`src/sparql/`**: SPARQL query execution and database extraction
- **`src/process.py`**: Dictionary processing pipeline

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [karlb/wikdict-gen](https://github.com/karlb/wikdict-gen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
