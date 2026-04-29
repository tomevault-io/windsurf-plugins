---
trigger: always_on
description: - Goal: build machine learning models for Kaggle's March Machine Learning Mania 2026 competition.
---

# AGENTS.md

## Project Overview

- Goal: build machine learning models for Kaggle's March Machine Learning Mania 2026 competition.
- The repository currently contains raw competition data in `data/` plus the local data dictionary in `data-description.md`.
- We only care about Stage 2.
- The solution must be LightGBM only, including any pooled, separate-division, single-model, or ensemble variants, and each inclusion must be justified by validation.
- The main prediction target is the submission format shown in `data/SampleSubmissionStage2.csv`.
- Submission rows use `ID=Season_LowerTeamID_HigherTeamID`, and `Pred` is the win probability for the lower TeamID team.
- The competition combines both men's (`M...`) and women's (`W...`) tournaments in one submission. Keep those data flows explicit so features are not mixed accidentally.
- Stage 2 requires predictions for every possible team matchup in 2026, not just the final tournament field.
- The local data description says the current-season files are complete through February 4, 2026. If a task depends on fresher competition data, verify whether the local files have been updated before modeling.
- Competition-host refreshes matter for current-season features: expect one final full refresh on Selection Sunday or the following morning, and treat late-week `DayNum=132` Massey updates as valid pre-deadline inputs when they become available.
- ONLY STOP/END THE TURN WHEN YOU REACH THE STOP CONDITION SET BY THE USER: As we get deeper into research, it's expected that we will have many iterations without progress in our metric. This is completely normal. Don't get discouraged by it and just look for new angles and unexplored approaches. 

## Evaluation And Validation

- Kaggle evaluates submissions with Brier score, which here is mean squared error on binary game outcomes.
- Local model selection should be driven by walk-forward validation by season.
- Always run the full walk-forward validation over the latest 5 available completed seasons.
- Use each year as its own validation fold with training restricted to prior seasons only.
- Validation-year features may be built for that season, but only from information that would have been available by that season's Selection Sunday or equivalent final pre-tournament data refresh.
- Report mean Brier across those 5 folds, and keep the individual fold scores for comparison.
- Any model, feature set, or ensemble must be accepted or rejected based on this validation.
- A candidate is only considered improved if it lowers mean validation Brier and improves at least 4 of the 5 folds.
- If a candidate lowers the average validation Brier but improves fewer than 4 of the 5 folds, discard it as not improved.
- Do not use random row splits for primary validation.
- Do not skip folds to iterate faster. Speed should come from caching and efficient feature generation, not from partial validation.
- Because 2026 Stage 2 submissions score `0.0` before the tournaments are played, do not use the Kaggle leaderboard for model selection before live games begin.
- Play-in games do not affect official scoring, so do not spend meaningful model-selection effort optimizing predictions for those matchups.

## Repository Layout

- `data/`: raw Kaggle CSV inputs. Treat these as immutable source data.
- `data-description.md`: competition data dictionary and submission semantics.
- There is currently no Python package, no test suite, and no build configuration in this repo. Do not invent extra project structure unless the user asks for it.

## Workflow Expectations

- Start with simple checks before deeper changes: schema inspection, season coverage, duplicate game keys, null handling, and row-count sanity checks.
- Prefer Polars `LazyFrame` pipelines for feature engineering and joins. Push work into the Polars engine instead of Python loops.
- Use `duckdb` or lazy CSV scans for large-file inspection. `data/MMasseyOrdinals.csv` is especially large, so avoid eager full-file loads unless necessary, especially when reprocessing updated `DayNum=132` snapshots during submission week.
- The local machine has 32 GB RAM and 4 CPU cores. Favor memory-aware transformations, cached intermediates, and reusable feature tables.
- Keep raw inputs untouched. If new output directories are needed, create user-facing locations such as `artifacts/`, `models/`, or `submissions/` only when required and keep large generated files out of git.
- Use `data/SampleSubmissionStage2.csv` as the canonical 2026 submission target.
- Maintain a root `JOURNAL.md` and append a short entry for each completed run that changes features, params, or ensemble composition.
- Default experiment runs should be validation-only. Do not generate a submission file or dump trained model files for every experiment.
- Accepted experiments should still be submission-capable in design, even when the script only runs validation.
- For any accepted candidate, keep a clear path to build the same feature set and probability mapping for the full 2026 Stage 2 submission universe once the required current-season files are available.
- Only create a full Stage 2 submission file and persist trained model artifacts when the user explicitly asks for them.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ledmaster/ml-mania-2026](https://github.com/ledmaster/ml-mania-2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
