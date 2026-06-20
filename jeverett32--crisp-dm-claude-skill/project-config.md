---
trigger: always_on
description: Builds a production-grade ML pipeline following a staged CRISP-DM framework. Uses a modular 'src/' + independent notebook architecture. Employs staged interviews to minimize hallucinations and gather comprehensive context per phase. Outputs individual self-contained phase notebooks AND a comprehensive, runnable Master notebook with an executive summary.
---


# CRISP-DM Staged Machine Learning Pipeline

This skill generates a complete, end-to-end ML pipeline grounded in the CRISP-DM framework (problem framing → data understanding → preparation → modeling → evaluation), and can optionally "operationalize" that pipeline into a repeatable training + inference workflow.

## Key Design Principles

1. **Staged Interviews**: The process is NOT a single initial interview. We pause after each phase to conduct a targeted interview for the next phase. This gives the agent richer context and less room to hallucinate.
2. **Modular Architecture**: All reusable logic (data loading, preprocessing, modeling, evaluation) lives in `src/`. Notebooks are thin interfaces that import from `src/`, ensuring consistency and independence.
3. **Independent Notebooks**: Each phase notebook AND the Master notebook must be self-contained and runnable independently. They check for/load intermediate artifacts or regenerate them if missing.
4. **Sign-off Gates**: Before proceeding to the next phase, the agent must present a "Phase Summary & Conclusion" for explicit user sign-off.

---

## Step 0: Initialize a Lab Tree (workspace for all outputs)

Before writing analysis/code artifacts, create a **lab workspace folder** in the repo so the project stays organized and repeatable.

### Lab tree rules
- Put **all generated code, notebooks, and reports under the lab tree** (do not scatter files at repo root).
- Do not modify existing application code outside the lab tree unless the user explicitly asks.
- Use a short, filesystem-safe project slug derived from the user's goal, e.g. `churn_prediction`, `late_delivery`, `house_prices`.

### Standard lab tree layout (create if missing)

```
lab/
  <project_slug>/
    README.md
    requirements.txt              # or pyproject.toml if project uses it
    data/
      raw/                        # immutable inputs (or links/notes)
      interim/                    # intermediate transforms
      processed/                  # modeling-ready tables
    notebooks/
      01_business_understanding.ipynb
      02_data_understanding.ipynb
      03_data_preparation.ipynb
      04_modeling.ipynb
      05_evaluation.ipynb
      master_crispdm_pipeline.ipynb   # Independent, stakeholder-ready
    src/
      __init__.py
      config.py                   # paths, constants, random seeds
      data_io.py                  # loading utilities
      features.py                 # feature engineering (shared by train + infer)
      metrics.py                  # metric helpers/baselines
      modeling.py                 # model definitions / tuning helpers
      evaluation.py               # evaluation plots/reports
    jobs/                         # operationalization (optional)
      etl_build_warehouse.py
      train_model.py
      run_inference.py
      utils_db.py                 # if DB source/sink is used
    artifacts/
      models/                     # saved model pipelines (.sav)
      runs/                       # per-run metadata/metrics
    reports/
      figures/
      tables/
      executive_summary.md
    logs/
```

If the user does not want operationalization, you can omit creating `jobs/` initially, but keep the lab tree structure so the work remains organized.

---

## Step 1: The Staged Interview Protocol

**CRITICAL**: Do NOT ask all questions up front. Follow this staged approach. The agent gathers context progressively, which leads to better-informed questions and less hallucination.

### Phase 0: Initial Intake (ask once, at the very beginning)

Extract what you can from the conversation context — only ask what's genuinely missing.

**Minimum viable set:**
1. **Goal & decision**: What decision will this model support, and who will use it?
2. **Data source & access**:
   - **Files**: path(s) (CSV/Excel/Parquet), or
   - **Database**: engine (SQLite/Postgres/etc), file/connection info, and table(s), or
   - **Multiple sources** that need joining/denormalizing?
3. **Target definition**:
   - Target column name (or how to construct the label)
   - If classification: what is the **positive class** (the "important" class)?
4. **Output choice**:
   - **Operationalization** (repeatable train + infer workflow): **Yes/No**
     - If Yes: where should predictions go? (file, database table, app integration point)

### The Phase Loop (Repeat for Phases 1-5)

For each CRISP-DM phase, follow this sequence:
1. **Interview**: Conduct the phase-specific deep-dive interview (see Staged Probing Schedule below).
2. **Implementation**: Generate/update the phase-specific notebook and relevant `src/` modules.
3. **Verification**: Present the phase results to the user.
4. **Sign-off**: Request explicit user sign-off on the phase conclusions before proceeding.

**Do not move to the next phase until the user has explicitly approved the current phase.**

### Staged Probing Schedule

| After This Phase | Conduct This Interview (for the NEXT phase) |
|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jeverett32/crisp-dm-claude-skill](https://github.com/jeverett32/crisp-dm-claude-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
