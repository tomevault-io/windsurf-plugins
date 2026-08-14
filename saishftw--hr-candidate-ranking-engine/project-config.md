---
trigger: always_on
description: Orientation for any AI agent picking up this repo in a **new session**. Read this first,
---

# AGENTS.md — AI Recruiter Pipeline

Orientation for any AI agent picking up this repo in a **new session**. Read this first,
then the linked docs as needed. Keep it concise — it is loaded into context each session.

## What this is
A candidate scoring/ranking pipeline for recruiting, plus an **evaluation harness** that
gauges and regression-guards accuracy. Primary client/data: **Prime Focus Group** (UAE
HVAC/manufacturing, "HR Assistant" role). Candidate pool = a LinkedIn export (145 profiles);
JDs are free text → structured `JobRoleSchema` via an LLM.

## Current state (champion)
Weights (renormalized when a component is inactive for a JD):
`title .25 / skill .25 / qualification .05 / similarity .45 / seniority .05 / experience .05 / industry .20 / language .15 / location .05 / attrition .005 / experience_relevance .015 / education_relevance .005`
(`language` .15 is measured on the tenure/relevance/language-aware silver Judge-grade anchor: NDCG@10
.9490→.9588, with reverse metrics unchanged after language de-leak. The three structural weights are tiny
**product-motivated tie-breakers** adopted after their standalone T3 sweeps rejected larger weights: joint
NDCG@5/10 remains .9464/.9588, NDCG@20 improves .9289→.9518, and reverse MRR .5190→.5212. Original-LLM
Spearman slips .2600→.2568; fit-NDCG@10/top-k overlap hold. Tagalog-specific evidence is n=1. `location` .05
remains product-motivated. Also shipped (C5/U2):
per-role `positions[]`, a **data-completeness flag**
(rich/partial/low → screening lane, NOT in `total_score`), and the **swipe-feed card** contract (`core/swipe.py`,
`scripts/build_swipe_cards.py`) — see `docs/specs/recruiter-signals-and-swipe-feed.md`, `docs/DECISIONS.md` §C5.)
Title gate = **hybrid** (max of fuzzy & semantic), **soft** (no hard drop). Skill match = **hybrid**
(max of fuzzy char-ratio & embedding cosine; semantic floor `skill_semantic_threshold = 0.40`).
Component min-max normalization = **OFF** (tested, it hurt). **Similarity embedding** = `Qwen3-Embedding-0.6B`
(isolated to `similarity_score`; title/skill keep `all-mpnet-base-v2`), adopted **product-motivated**
(all-mpnet truncates ~44% of profiles at 384 tokens; the n=1 short gold JD can't measure the long-context
gain) — instruction-prompted, fp16, `max_seq_length=1024` (`models/mappings.py:similarity_model_config`;
set it to `None` to revert to all-mpnet). Truncation is now logged whenever inputs exceed the cap.

Honest metrics (de-leaked reverse-match + one real-JD **silver** Judge-grade anchor, n=1; Qwen champion):
- reverse: MRR **.5212**, hit@3 **.579**, hit@5 **.632**, hit@10 **.790**, seed_found 1.0
- silver anchor: NDCG@10 **.9588** / NDCG@5 **.9464** / NDCG@20 **.9518**
- two-judge agreement over the frozen 78-candidate cohort: overall Spearman **.922**; tenure **.650**;
  career relevance **.884**; preferred signals **.813**. Pipeline NDCG@10 before C5 reblend **.949** vs
  original LLM shortlist **.686**. Still circular + n=1; recruiter swipes/U2 are the un-circular check.

## How to run (uv, Python 3.11)
- Full offline eval + regenerate baseline:
  `COPILOT_SKIP_CLI_DOWNLOAD=1 uv run python scripts/run_eval.py --dataset linkedin --n-per-group 5 --out evals/results/baseline_linkedin.json`
- Tests: `COPILOT_SKIP_CLI_DOWNLOAD=1 uv run pytest tests -q --ignore=tests/test_eval_pipeline.py`
  (the ignored test hits the network / live LLM.)
- Weight tuning (offline, exact-parity): `scripts/calibrate_weights.py` — `--ablate <comp>`, `--redundancy A B`, `--joint`, or `--c5-reablate`; skill-matcher ablation: `--skill-mode {fuzzy,semantic,hybrid} --skill-semantic-threshold X`
- Real run on the HR JD + comparison to the LLM-scored pool: `scripts/run_hr_assistant.py`
- Blind LLM adjudication (live Copilot): `scripts/blind_judge_rankings.py`
- Copilot connectivity smoke test: `scripts/smoke_copilot.py` (verify auth/runtime before a live run)
- Model latency + blind-quality benchmark (JD-gen task): `scripts/bench_models.py`
- `COPILOT_SKIP_CLI_DOWNLOAD=1` keeps Copilot imports offline (tests/offline sweeps). **Unset it** for live LLM calls.

## How to test / ablate a new change
The **ablate-then-adopt loop**. Never adopt a knob/component without measuring it on the
leakage-free anchor (**gold NDCG@10**) and clearing the regression floors. Exact commands are in
*How to run* above.

1. **Implement** behind the existing seams. New component → add `calculate_<x>_score(df, jd)` in
   `core/scoring.py`, its weight in `candidate_score_weights` (`models/mappings.py`), make it
   **active-gated** inside `calculate_total_score` (it counts only when the JD supplies that signal),
   wire it into `core/pipeline.py` + `evals/runner.py` (same scorer order), and add
   `tests/test_<x>_score.py`. Tuning-only change (weight, threshold, title/skill mode) → just edit the knob
   in `models/mappings.py`.
2. **Ablate offline** with `scripts/calibrate_weights.py` (precomputes the component columns once,
   then re-combines cheaply — no LLM):
   - `--ablate <comp>_score` — 1-D weight sweep for the new component
   - `--redundancy <A>_score <B>_score` — is it complementary or redundant vs an existing one?
   - `--joint` — re-validate the core mix (sweeps title/skill/qual/similarity/seniority/experience;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [saishftw/hr-candidate-ranking-engine](https://github.com/saishftw/hr-candidate-ranking-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
