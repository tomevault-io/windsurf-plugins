---
trigger: always_on
description: This project has a design (`DESIGN.md`) and is now **v1-released**: Phases
---

# CLAUDE.md — Context for AI Assistants

This project has a design (`DESIGN.md`) and is now **v1-released**: Phases
1-4 (§10) are complete with real results, `README.md` is published, and the
repo is git-tagged `v1`. This file records **why** the design looks the way
it does, so a fresh session doesn't re-litigate settled decisions or
re-propose rejected ones.

Read `DESIGN.md` first, then `README.md` for the actual results. This file is
the reasoning behind the design.

---

## Current status (2026-09-09, v1 tagged + audited, Phase 5 appendix built)

**Environment:** venv named `expedia` (not `.venv`) at the project root —
`source expedia/bin/activate`. Package installed editable (`pip install -e .`)
as `ranking`, importable from `src/ranking/`. Both datasets downloaded and
extracted under `data/expedia/` and `data/trivago/`; `data/processed/` fully
materialised. `docker` + `docker compose` present and verified working.

**Done — Phases 1-5.** Numbers below are post-audit (an independent review
found and this repo fixed a broken calibration model, a corrupted propensity
curve, a SNIPS rank/position bug, and a `.gitignore` that excluded the entire
`src/ranking/models/` package from git — see the audit commit for the list).
- Phase 1: arms 0/1/1b/2/3 on both datasets — `results/phase1_expedia.md`,
  `results/phase1_trivago.md`.
- Phase 2 (Deliverable A): full-scale propensity/IPW/SNIPS/calibration run —
  `results/phase2_expedia.md`. Headline: IPW beats naive LambdaMART by
  **+0.0028 NDCG@38** [+0.0002, +0.0053] on the sealed randomised holdout —
  real but modest: MRR agrees (+0.0038 [+0.0004, +0.0078]), NDCG@10 does not
  clear significance, and SNIPS (independent estimator) agrees on direction.
- Phase 3 (Deliverable B): session ranking arms, retrieval coverage over the
  full 927k catalogue, MMR re-ranking — `results/phase3_trivago.md`. Session
  features are the biggest lever (MRR 0.47 → 0.61); the fully-stacked ranker
  (session + kNN + SASRec) is slightly *below* session-features-alone — an
  honest null result, kept rather than hidden.
- Phase 4: promotion gate simulation (`results/promotion_gate.md`) and load
  test against the live serving stack (`results/loadtest.md`).
- `README.md`: results tables, architecture diagram, `docker compose up`
  reproduce (verified end-to-end), latency/fallback numbers, shipping
  recommendation, limitations.
- Phase 5 (appendix, post-v1 by design): **arm 6 LightGCN**
  (`results/lightgcn.md`, `src/ranking/models/lightgcn.py`) and the **RQ-VAE
  semantic-ID tokenizer** (`results/semantic_ids.md`,
  `notebooks/semantic_ids.ipynb`, `src/ranking/rqvae.py`) — both labelled as
  learning artifacts, neither producing a shipping recommendation. Two
  findings there were NOT predicted and are the reason the arm earned its
  place: on cold-positive clickouts LightGCN actively promotes the warm items
  it knows above the correct one (worse than displayed order 19% of the time,
  i.e. "confident wrong opinion", not "no opinion"); and training it 5x longer
  improved its own BPR objective +47% while making the ranking metric worse,
  which is a train/eval objective mismatch (uniform catalogue negatives vs
  discriminating within a ~25-item impression list), not a model-class ceiling.
- 63/63 tests pass (`pytest tests -q`), no warnings.

**Two real bugs found and fixed while wiring Phase 2-4 together, worth
knowing about if touching serving code:**
1. `serving/pipeline.py::build_frame()` never computed `knn_score`/
   `sasrec_score`, so the originally-configured primary model
   (`lambdamart_full`, which needs those features) silently failed every
   request and fell back to item-kNN even when "healthy". Fixed by making
   `lambdamart_session` the default `PRIMARY_MODEL` (it's also the
   better-measured Phase 3 arm) rather than adding live kNN/SASRec scoring
   to the hot path. `lambdamart_full`'s online-serving gap is documented in
   README as a known limitation, not hidden.
2. `docker compose up` had never actually been exercised: `python:3.11-slim`
   is missing `libgomp1` (LightGBM's compiled booster needs it), and
   `README.md` didn't exist yet even though the Dockerfile `COPY`s it. Both
   fixed; a full build+up+`/rank` round-trip is now verified working.

**Remaining — optional, post-v1 per DESIGN.md's release gate:**
- Phase 5 (appendix): LightGCN warm-subset confirmation, RQ-VAE notebook.

## What this is

A recommender-systems portfolio project for **applied DS / MLE roles at travel and
e-commerce companies** (Expedia, Booking, Flipkart, Amazon).

Two datasets, two self-contained deliverables:
- **Expedia ICDM 2013** → unbiased learning-to-rank (position-bias correction)
- **Trivago RecSys 2019** → session-based ranking + a production serving layer

## Who it's for

Solo developer, targeting industry hiring — **not** academic publication.
Hardware: single 8GB RTX 5050. Time is not a hard constraint; **motivation over a
long project is the real risk.** Prior project (`../pho2rec`) stalled at step 5 of
14, which is why the design has a v1 release gate.

Demonstrated existing strengths, all of which port to this project: FastAPI, async
SQLAlchemy, PostgreSQL, Redis, React/TypeScript, OpenCLIP embedding pipeline, a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Reeptide/propensity](https://github.com/Reeptide/propensity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
