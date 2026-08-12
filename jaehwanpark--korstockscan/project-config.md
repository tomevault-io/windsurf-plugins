---
trigger: always_on
description: KORStockScan working rules:
---

KORStockScan working rules:

## 1. Required References

- Before starting work, read `docs/plan-korStockScanPerformanceOptimization.rebase.md` §1 through §8 and the two top summary sections of the current daily `docs/checklists/YYYY-MM-DD-stage2-todo-checklist.md`: the daily objective and the daily mandatory rules.
- Use `Plan Rebase` as the source of truth for tuning principles, decision axes, rollback guards, and active/open state. The daily checklist owns executable work items.
- The clean tuning data decision baseline is `clean_tuning_baseline_date=2026-06-05` and `clean_tuning_baseline_ts_kst=2026-06-05T00:00:00+09:00`. Pre-baseline raw/report/analytics artifacts are archive/audit evidence only and must not be used for EV, rolling/MTD/cumulative tuning, live-auto promotion, runtime approval, pattern lab promotion, or real execution quality approval. The policy artifact is `data/source_quality/clean_baseline_policy.json`.
- Read the current active/open state from `Plan Rebase` §7 through §8. Do not treat completed `[x]` items in older checklists as current OPEN owners. Completed items are evidence links. The current owner is the runtime owner named on the same row or the open item in the current checklist.
- `docs/plan-korStockScanPerformanceOptimization.prompt.md` is a lightweight session-entry pointer. Do not read it for every routine task. Check it only when the user explicitly asks, or when the Plan Rebase location, Source of Truth document map, or current execution table is unclear.

## 1.1 Current State Baseline (`2026-08-11 KST`)

- The current phase is the automation-chain tuning phase in `Plan Rebase`. The objective is not loss suppression, but expected value and net-profit maximization. Target-date PREOPEN runtime env or exact-date policy artifacts own selected runtime families. Active operating families include `soft_stop_whipsaw_confirmation`, `scalp_sim_candidate_window_expansion`, `scalp_sim_ai_budget_manager`, `lifecycle_decision_matrix_runtime`, `scalp_sim_auto_approval`, `swing_sim_auto_approval`, and the separate `samsung_machine_entry_policy` bounded PREOPEN family; blocked runtime-apply bridge families such as `entry_wait6579_score66_69_recovery_gate_v1` are not active runtime selections. Samsung morning remains baseline-only, while midday/afternoon can tighten at most one machine and one entry axis per next-session candidate without changing quantity, target, validity, no-stop holding, provider/bot/cap, or broker guards.
- Tuning data decisions now start from the clean baseline `2026-06-05 KST`, with full-day start timestamp `2026-06-05T00:00:00+09:00`. Raw/report/analytics data before that baseline is archive-only and cannot support EV, rolling/MTD/cumulative tuning, live-auto promotion, runtime approval, pattern lab promotion, or real execution quality approval. `threshold_cycle_preopen_status` and `threshold_cycle_postclose_status` are operational freshness status artifacts, not tuning decision reports, and are excluded from clean report quarantine.
- Postclose `observation_source_quality_audit` is now the contract-gap preflight hard gate for tuning inputs. Contract warning/fail, required-field missing, invalid label, or high-volume no-contract gaps set `tuning_input_allowed=false`; EV, rolling/MTD/cumulative tuning, live-auto promotion, and runtime approval must become `source_quality_blocked` with `runtime_effect=false` and a `source_quality_gap` code-improvement workorder handoff. Unknown-token findings alone are review warnings, not immediate tuning stops.
- The central loop is `R0_collect -> R1_daily_report -> R2_cumulative_report -> R3_manifest_only -> R4_preopen_apply_candidate -> R5_bounded_calibrated_apply -> R6_post_apply_attribution`. Artifact, consumer, and apply contracts are owned by `docs/report-based-automation-traceability.md`.
- The scheduled automation-chain path is postclose artifact plus AI/deterministic guards and PREOPEN `auto_bounded_live` runtime env. An explicit user-directed intraday override may change one existing `bounded_tunable` axis for one explicit KRX or NXT cohort only with fresh conflict-free source quality, effective price, target-first or sufficient post-block MFE, a single-blocker causal link, no same-stage owner/canary conflict, recorded before/after values and runtime provenance, rollback, and immediate attribution. It must not alter hard safety, stale/conflict, price freshness, broker/account/order/quantity/cooldown, provider, bot, cap, requested quantity, or a common/KRX surface from NXT evidence. Closed operational override records are archive evidence only; current owner decisions retain only selected family, provenance, and post-apply attribution.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JaehwanPark/KORStockScan](https://github.com/JaehwanPark/KORStockScan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
