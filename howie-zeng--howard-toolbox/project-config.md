---
trigger: always_on
description: - Do not delete user comments in code when editing; preserve all existing comments
---

# AGENTS.md

## Learned User Preferences

- Do not delete user comments in code when editing; preserve all existing comments
- Do not wrap code in tryCatch or other defensive error-handling unless explicitly requested
- Assume required columns/variables always exist; do not write column-existence checks or fallback logic (e.g., `if ("col" %in% names(dt))`)
- Undersampling belongs in model_run scripts, not dataprep scripts; dataprep should output the full dataset
- Explain reasoning and present a plan before changing code; do not make changes without approval; never add items not in the plan — user will ask for double-checks
- Use raw (uncapped) variable names for report continuous vars (e.g., `cur_factor` not `c_cur_factor`, `dscr_ratio` not `c_dscr`) so reports show actual data distributions
- Only add model features that are statistically significant; do not add variables just because they seem relevant
- Store SQL queries in separate files from analysis scripts; put ad-hoc analysis under `Residential/non_qm/adhoc/`
- When presenting model output, pass only one `model_list` at a time to report functions; for insample reports use `ed_fit_full` (all data) with Begg normalization applied, dropping excluded states; default `use_parallel = TRUE` in optimized reporting where supported
- Never edit production model files in place; copy to a local gitignored working directory first and write edits as separate files (e.g., smooth notebook copies the N-drive prod JSON into `smooth/local_models/` and writes `*.smooth_edit.json` outputs alongside)
- "Revise for management" means conclusions/takeaways first, less technical jargon, executive-style bullets with summary at top; internal emails should sound natural and like the user, using full forms such as "I will" rather than "I'll"
- Usage analyzer "Usage Volume" section must keep tokens as the primary signal; prefer one chart with dual axes (daily token bars left, 7-day request average line right) over removing tokens

## Learned Workspace Facts

- NQM CtoP uses Begg competing-risk (Cto0, CtoM3, CtoC); split into turnover (non-positive incentive) and refi (positive incentive) sub-models; scripts under `C/CtoP/script/1.4/Fixed/` with shared `C/` config/dataprep; `add_nqm_model_features()` `model_type` must match the model ("turnover", "refi", etc.) — there is no "Cto0" branch; CtoC report needs `annulize = FALSE` (monthly ~97-98% annualizes to flat 100%)
- Non-QM LLPA uses a GAM (not a static FICO x LTV grid); doc groups FULL (Full Doc + Tax Returns), BANKSTAT, DSCR, ALT (Asset Depletion + VOE), PL_CPA (P&L/CPA), OTHER; LLPA on lag1 rates; SATO and incentive use lag0/lowest-weekly; export via `build_sim_model_from_list()` with adaptive smooth (`bs='ad'`) auto-skip; time smooth endpoints appended per doc group for C++ decay; 2 SD residual trimming standard for SATO
- Incentive pipeline is spread-based: `inc_0_spread_llpa`, `sato_lag0_spread_llpa`, `burnout_lag0_spread_llpa`; `full_index_spread_fade15` replaced `full_index_inc_fade15`; turnover incentive capping targets `c_inc_0_spread_llpa`
- Turnover prepayment penalty: `inc_regime_f` includes `pp_active_Y`; `new_m2ppexp_f` bins months-to-PP-expiry for `pp_penalty == 1` (No PP plus ordered buckets through 4+); NQM burnout window 60 months (`burnout_inv_b2_60`); `inc_fade_period` 36; seasonality categorical (`month_f`); prefer `months_since_m30p` over `mc_age_ratio`; exclude COVID 2020-02–2020-06 from insample reports; foreign national `lmt.foreign` 0–3, not binary
- Model extract pipeline: `build_sim_model()` / `build_sim_model_from_list()` export R models to C++ text; `variable_mapping_{product}_test.csv` maps R names to C++ names; CSV OLD column needs spaces stripped (not `make.names()` — `check.names=FALSE` preserves curve column names through assembly/write/read); `verify_model_txt()` validates all entries post-extraction; extract sources `C/config.R` for model paths; production C++ scoring uses logit (sigmoid), so align R GAM/link with logit rather than cloglog where outputs feed that pipeline
- Non-QM data quirks: Fixed Balloon loans are ARM (`fix_f = "ARM"`) via NON_QM_FIX_MAP but often margin=0 — treat margin=0 as missing for ARM reset calculations; DSCR doc_map: loans with `dscr_ratio` should be "DSCR" when doc_map is NA, "NO DATA", or "OTHER"; `dscr_valid_f` and `doc_type_2` can be near-collinear in GAM (bam contrast errors)
- Data sources: loan-level `lps.lp_stat`, `lps.lp_dy`, `lps.lp_loss`; NQM rates `Libremax..MonthlyNQMRate`; HPI joins for inflation-adjusted o_bal. LMSim NQM monthly flatfiles under `N:\FlatFilesMonthly\NQMMonthly` include generated pseudo/stratification files; when deriving actual pool IDs or combine inputs, filter by the `_YYYYMMDD` `asof_date` suffix and exclude prefixes like `NQM_TOP`, `NQM_ALL`, `NQMAGE`, `NQMDOCTYPE`, `NQMFICO`, `NQMPRODTYPE`, `NQMUPDLTV`, and `NQMWAC`. Explicit deal-name filters should query `CRTStats` directly by `bbg_deal_name`, then separately warn/skip deals with no monthly flat file. LMSim `lm_sim_pub_main.py` no longer accepts `-dbsavefromdll_json*`; use `-purpose`, with forward projection purpose as `{purpose}_tracking`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [howie-zeng/howard-toolbox](https://github.com/howie-zeng/howard-toolbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
