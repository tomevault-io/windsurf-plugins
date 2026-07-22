---
trigger: always_on
description: This repo runs benchmarks against gbrain. It depends on `gbrain` as a library
---

# gbrain-evals — repo guide

This repo runs benchmarks against gbrain. It depends on `gbrain` as a library
(via the GitHub URL in `package.json`). Eval corpora and adapters live here so
gbrain itself stays small and benchmarks evolve independently.

## Repo layout

- `eval/runner/` — one `.ts` per benchmark family (cat13b-source-swamp,
  longmemeval, etc). Each runner produces a JSON in `eval/reports/` and an
  optional markdown summary.
- `eval/data/` — committed corpora (world-v1, source-swamp-v1, gold qrels,
  amara-life-v1) + content-addressed embedding caches. Anyone who clones gets
  warm fixtures.
- `eval/reports/` — gitignored. Transient run output. Baselines worth keeping
  get hand-copied into `docs/benchmarks/<slug>/`.
- `docs/benchmarks/` — published reports + the SVG charts they reference.
- `node_modules/gbrain` — symlink (when locally linked) or npm fetch (when
  pinned to a SHA in package.json). Link a local checkout for development:
  `cd ~/git/gbrain && bun link && cd ~/git/gbrain-evals && bun link gbrain`.

## When you write a benchmark report — the platonic ideal

The published report at `docs/benchmarks/<date>-<slug>.md` is the artifact a
reader stumbles into who has never heard of gbrain or the benchmark we're
running. **It must stand on its own.** No "see our docs," no "as we explained
in PR #X," no internal jargon without a gloss.

Every benchmark report MUST have these sections, in this order:

### 1. Headline (one viewport)

- Bold one-sentence verdict with the comparison: "gbrain hits 99.X% on the
  public LongMemEval `_s` split, X points above [closest published system]."
- A single SVG card showing the head-to-head numbers across systems.
- A short paragraph (2-3 sentences) naming what changed and why a reader
  should care.

### 2. What is gbrain

3-4 paragraphs of plain prose. Write for someone who has not heard of gbrain
before and has 60 seconds. Cover:

- Personal-knowledge brain that runs locally, no cloud lock-in, files on
  disk + Postgres index. Markdown source of truth, derived index.
- Hybrid retrieval: keyword + vector with RRF fusion + source-aware boost
  + optional Haiku query expansion. Each layer earns its keep on real
  retrieval workloads.
- "What's it for" — capturing notes, contacts, deals, decisions; recalling
  them weeks later when the context is gone; never losing the connection
  between two things you wrote down apart.

Link to the gbrain repo. Link to the gbrain CLI README. Don't repeat the
README; just give a reader enough to know whether they're in the right
neighborhood.

### 3. What is the benchmark

For LongMemEval and ConvoMem and any future public benchmark we support:

- Who built it. Link to the paper / HuggingFace / GitHub.
- What's in the dataset. Question count. Question types. Ground-truth shape.
- What metric we report (Recall@k, R@k, QA accuracy, etc.) and why we
  picked it. If we picked retrieval recall over end-to-end QA, say so and
  link the published QA evaluator.
- "Why this benchmark" — what failure mode does it stress that other
  benchmarks don't.

### 4. Adapters tested — every gbrain feature explained

For EVERY gbrain configuration in the comparison table, write a short
section that includes:

- **What this adapter is.** One sentence describing the retrieval pipeline.
- **What gbrain feature it exercises.** Name the actual code path:
  `engine.searchKeyword`, `hybridSearch`, `expandQuery`, the source-boost
  CASE expression in `sql-ranking.ts`, etc.
- **Why it would matter on this benchmark.** Tie it to the kind of
  question that exercises this layer.
- **Real-world use case.** What does this look like in a user's actual
  workflow? "Recalling a preference you stated three months ago" is a
  better example than "single-session-user question type."

Example for hybrid+expansion:

> **`gbrain-hybrid+expansion`** runs `hybridSearch(engine, query, {expansion: true, expandFn: expandQuery})`. The keyword + vector RRF stack from `gbrain-hybrid` plus a Claude Haiku call that rewrites the user's question into 2 alternative phrasings. All 3 phrasings hit the index; results fuse via RRF. This is gbrain's CLI default (`gbrain query` ships with expansion on).
>
> **Why it helps:** the question often uses different vocabulary than the answer. "What car issue did I mention?" doesn't share words with "the GPS isn't working right after my dealership visit." Vector embedding closes some of that gap; query expansion closes more by generating alternative phrasings the vector model can match.
>
> **Real-world parallel:** when you ask gbrain "who do I know who works in vertical AI" it doesn't just match "vertical AI" verbatim. Haiku expands to alternative phrasings like "AI for specific industries," "applied AI startups in narrow domains" — then RRF-fuses. Without expansion, narrow phrasings hide what's in the brain.

### 5. Results — head-to-head table

Required columns:
| System | Recall@k | k | n | LLM in retrieval loop | Source |

Include EVERY published system we can find at the same metric. MemPalace,
Stella, Contriever, BM25 all publish numbers on LongMemEval R@5;
they belong in the table. Mastra and Supermemory publish QA accuracy not
R@k — flag them as "different metric, not directly comparable" but keep

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [garrytan/gbrain-evals](https://github.com/garrytan/gbrain-evals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
