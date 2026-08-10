---
trigger: always_on
description: Guidance for Claude Code (claude.ai/code) when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) when working in this repository.

## Goal

Compare **self-hostable memory-plugin providers for the Hermes agent** on the
[MemConflict](https://github.com/TaoZhen1110/MemConflict) benchmark. Each provider
runs in a **best-effort configuration** a real Hermes deployment would plausibly
use, not a hand-tuned lab setup.

Providers: **Mnemosyne** (reference), **Hindsight**, **mem0**, **Supermemory**,
**Honcho**, **OpenViking**, **RetainDB server edition**. Status per provider,
plus all measured results and per-arm numbers, is in `docs/BENCHMARK_MATRIX.md`.

**~~RetainDB Local~~ (npm `@retaindb/local`) is RULED OUT (2026-07-22):**
`search()` rebuilds the concept graph per candidate, so search is O(n²), ~245
s/query at n=2,897, no vendor knob, no banked number
(`docs/TROUBLESHOOTING.md`). `retaindb_server/` is a different product and is not
covered.

### The three standing rulings

**1. Fairness lives in the shared harness.** Every provider runs the same
dataset, answer model, judge model, top-K, and provider-agnostic scorer. A change
to that shared harness which moves one provider's numbers invalidates the
comparison. Provider-internal configuration is not that line.

**2. Best-effort configuration, applied evenly** (user, 2026-07-21). Tune every
provider's vendor-exposed knobs, not only the one that visibly misbehaves. Set
`MNEMOSYNE_LLM_MAX_TOKENS=3072`, `HINDSIGHT_API_LLM_TEMPERATURE_RETAIN=0.7`. A
hardcoded value with no knob is a property of the product. Prefer values the
vendor or model card endorses and record the justification. "We tuned until the
number went up" is out of bounds.

**3. Measure what the plugin returns** (user, 2026-07-21). The unit of measurement
is what each provider's Hermes plugin hands the agent at recall time. If a plugin
surfaces extracted facts rather than dialogue turns, facts reach the answer model.
Do not reshape retrieval output to look more like raw evidence.

**The trap behind ruling 3.** The judge scores supporting evidence hit
**semantically**. It ranks the first retrieved memory whose evidence supports
the reference answer, told to "not require exact wording". Extraction and
paraphrase carry no penalty, so storage format is not a confound. A provider's
real retrieval output scoring worse is a finding to report.

## Repository layout

```
benchmark/        shared, provider-agnostic harness (scorer, summarizer, llm glue, docker)
docs/             DECISIONS.md, TROUBLESHOOTING.md, BENCHMARK_MATRIX.md
mnemosyne/  hindsight/  mem0/  supermemory/  honcho/  openviking/  retaindb_server/   adapters
retaindb/         RetainDB local-edition adapter (RULED OUT, kept for the record)
external/         submodules: MemConflict, mnemosyne, RetainDB, honcho, hermes-agent
```

**Dependency invariants:** every provider and harness folder sits exactly one
level under the repo root. Adapters resolve the dataset as
`../external/MemConflict/...` and write to file-relative `Results/` and `Scores/`.
Nesting a provider deeper, or moving `external/`, breaks those paths.

**Per-contract artifact folders.** Past artifacts live under
`<provider>/{Results,Scores}/v<N>/`, plus `unclassified/` for off-contract or
unverifiable smokes. Adapters write NEW runs to the `Results/` and `Scores/` root;
sort a run into `v<N>/` when you bank it. Classify by evidence (manifest
`git_sha` mapped to a compose checkpoint, or a `serving_envelope_*.json` sidecar),
never by run tag. The mnemosyne run tagged `v2_baseline` is contract **v3**.

**Three docs, no new ones.** `BENCHMARK_MATRIX.md`: providers, arms, flags,
results, planned runs. `DECISIONS.md`: why the harness is built this way,
including reversed decisions. `TROUBLESHOOTING.md`: symptom, cause, fix, what did
not work. The env-var surface stays in `benchmark/docker/README.md`.

## Harness contract

`<provider>/eval_<provider>.py` ingests each persona's multi-session dialogue into
an isolated store. Per question it emits one JSONL row with `Model_Answer` and
`Retrieved_Memories` (`memory`/`created_at`/`score`) into `<provider>/Results/`.

Scorer and summarizer are **provider-agnostic** and score any provider's
output:

- `benchmark/score_resumable.py`: resumable LLM-judge scorer, the primary scoring
  stage; a thin wrapper over `external/MemConflict/Evaluation/eval_scoring.py`
  through a `MEMCONFLICT_EVAL_DIR` `sys.path` insert.
- `benchmark/summarize_scores.py`: per-persona scores to headline metrics.
- `benchmark/llm_reasoning.py`: reasoning-effort and JSON-mode wrapper over the
  upstream `llm_request`. **The one cross-folder Python dependency:** every
  adapter does `from llm_reasoning import ...` and prepends `../benchmark` to
  `sys.path`. Keep that insert if you touch an adapter's import block.

### Metrics

Headline: **macro answer accuracy**, the unweighted mean over the three conflict
types (**dynamic, static, conditional**). Also reported: micro answer accuracy,
supporting evidence hit at K, log-rank@3, update order recognition, contradiction
recognition. Every summary carries the `by_conflict_type` split. Supporting
evidence hit is the judge's semantic measure, so it, the evidence utilization gap,
and log-rank all move with a judge-config change.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EngTurtle/hermes-memconflict](https://github.com/EngTurtle/hermes-memconflict) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
