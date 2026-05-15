---
trigger: always_on
description: Read on a fresh session. Procedural how-to for the adamsreview plugin. Reference docs load on demand: `docs/state-and-gates.md` (state model, score gates, lanes), `docs/pipeline.md` (phase trees + token-tally semantics), `docs/helpers.md` (helper inventory). `bin/schema-v1.json` is the source of truth for artifact shape. `docs/archive/` is frozen historical reference — consult only for past-decision rationale.
---

# CLAUDE.md — operational guide for adamsreview

Read on a fresh session. Procedural how-to for the adamsreview plugin. Reference docs load on demand: `docs/state-and-gates.md` (state model, score gates, lanes), `docs/pipeline.md` (phase trees + token-tally semantics), `docs/helpers.md` (helper inventory). `bin/schema-v1.json` is the source of truth for artifact shape. `docs/archive/` is frozen historical reference — consult only for past-decision rationale.

## What this repo is

Build repo for six Claude Code slash commands packaged as plugin `adamsreview`:

- `/adamsreview:review [--ensemble]` — multi-lens review. Phase 0 preflight → 1 detection (up to 6 parallel lens agents — L2/L5/L6 conditional on `trivial_mode` and user-facing flags — plus L7 under --ensemble in non-trivial mode; L7 is a holistic Opus safety net) → 1.5 external-source pooling (Codex CLI + GitHub bot-comment scrape, Sonnet normalizer) → 2 dedup → 3 cheap scoring + gate → 4 validation (Opus deep / Sonnet light chunked) → 5 cross-cutting (deep-lane only) → 5.5 auto-fix-hint generation (Sonnet propose+verify two-pass) for confirmed_manual / confirmed_report / light-lane confirmed_mechanical findings ≥60 → 6 finalize + render + publish. `--ensemble` adds the Codex CLI + PR bot-comment scrape sources.
- `/adamsreview:codex-review [--effort low|medium|high|xhigh]` — Codex peer to `:review`. Same phase set (0-6 plus Phase 5.5 auto-fix-hint generation), Codex-driven detection/validation/cross-cutting via `bin/codex-poll.sh`; Phase 1.5 skipped (no ensemble); Phase 5.5 is Sonnet-driven (validator-agnostic, shared with `:review`). Same artifact shape (`reviewer_sources: ["internal-codex"]`) so drop-in for `:fix` / `:add` / `:walkthrough` / `:promote`. Sonnet shape-fixers per Codex output; Codex tokens billed externally (only Sonnet shape-fixer + normalizer + Phase 5.5 in `tokens.jsonl`). Fail-fast on missing Codex (no Claude fallback).
- `/adamsreview:add` — inject external findings (cloud `/ultrareview` paste, Opus once-over, manual finds) into the latest review's artifact. Locate latest → leftover-attempted gate → build candidates (paste-normalizer | structured `--file/--line/--claim` | mixed) → Sonnet dedup → assign continuing IDs → Phase 4 lane-aware (no Wave 2) → re-publish to existing PR comment.
- `/adamsreview:walkthrough [threshold]` — interactive driver for findings `:fix` would skip (deep-manual, deep-report, full light lane), restricted to effective score ≥ threshold (default 60). Two scope tiers (Qualifying / Full skip set) — `pre_existing_report` always routed to end-of-run `gh issue create` flow. Step 4.5 batch-accepts findings carrying `auto_fix_hint` (set by Phase 5.5) before the per-finding loop; per-finding loop skips the briefer sub-agent when `auto_fix_hint` is present, reusing the pre-computed hint.
- `/adamsreview:fix [threshold] [--granular-commits]` — automated fix loop for auto-fixable findings. Phase 7 load + leftover-attempted abort + clean-tree gate + staleness check → 7.5 auto-recommendation preflight (surfaces `auto_fix_hint` findings for batch-accept; promotes via `human_confirmation` bypass so Phase 8 picks them up) → 8 per-fix-group agents (no git ops) → 9 post-fix Opus review, revert regressions, commit survivors, append `fix_attempts`. Phase 9.pre overlap → reconcile/abort/inspect. Default one combined commit; `--granular-commits` does one per surviving fix group.
- `/adamsreview:promote <id>` — metadata-only override. Promotes a single finding to auto-fixable, bypassing Phase 8 lane filter and threshold. Run `:fix` after.

Recommended flow on a non-trivial PR: `:review` (or `:codex-review`) → optional `:add` → optional `:walkthrough` → `:fix`. Each command is independent; `:promote` is for one-off promotes outside the walkthrough flow.

Detailed phase trees and the `subagent_tokens` / `orchestrator_tokens` semantics: `docs/pipeline.md`. Both fields are re-tallied before every lifecycle command's final re-render so the published PR comment shows cumulative spend across the review → fix / add / walkthrough arc; `orchestrator_tokens` is opt-in via `ADAMS_REVIEW_TALLY_ORCHESTRATOR=1` (macOS provenance-prompt avoidance).

Per-stage history: `plans/`. Historical backlog: `plans/old-backlog.md` (frozen 2026-05-04). Active follow-ups: GitHub issues.

## State, gates, lanes (TL;DR)

- **States.** `open` → `attempted` (Phase 8) → `resolved` (Phase 9 verified) | `→ open` (Phase 9 partial/regression). Leftover `attempted` on a fresh `:fix` → hard abort.
- **Disposition** is the routing key (11 values, set by Phase 3 / 4 / 9). `is_actionable` derives from disposition; never set independently.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adamjgmiller/adamsreview](https://github.com/adamjgmiller/adamsreview) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
