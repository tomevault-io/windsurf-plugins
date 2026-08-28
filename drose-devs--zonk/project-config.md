---
trigger: always_on
description: This file is the codex-specific execution guide for this repository.
---

# zonk AGENTS (Codex Working Instructions)

## Scope

This file is the codex-specific execution guide for this repository.
It complements `CLAUDE.md` with mandatory constraints for automated work.

## Core rule

- Autoresearch loops MUST be implemented and executed in Rust.
- Do not use Python for looping or orchestration of candidate generation, backtest runs, scoring, or ranking.
- This repository’s automated discovery workflow is paper-research-first: use arXiv/Exa seeds, then execute as `paper-research`.

## Zonk architecture at a glance

- Binary strategy runner: `zonk` (`src/main.rs`, `src/lib.rs`)
- Core CLI: `src/cli.rs`
- Core data pipeline: `src/data/*`
- Strategy implementations: `src/strategies/*`
- Metrics: `src/metrics/*`
- Autoresearch loop binary: `src/bin/autoresearch_loop.rs`

## Rust autoresearch command

- Build:
  - `cargo build --release`
- Run:
  - `cargo run --release --bin autoresearch_loop -- --seed-web --candidates 100 --top 10 --verbose`
  - `cargo run --release --bin autoresearch_loop -- --seed-web --verbose`
  - Optional dates/sessions override:
    - `--train-start 2020-01-01 --train-end 2024-12-31 --test-start 2025-01-01 --test-end 2026-03-11 --train-sessions 1008 --test-sessions 252`
  - Optional binary override:
    - `--zonk-bin target/release/zonk`

### Iterative refinement (default)

Looping is the default (`--max-rounds 10`). After round 0, the loop refines top winners by perturbing parameters within discrete grids and swapping assets. Stops on convergence (`--patience 3`, `--min-improvement 0.02`) or frontier exhaustion.

| Flag | Default | Description |
|------|---------|-------------|
| `--max-rounds` | 10 | Maximum refinement rounds |
| `--patience` | 3 | Stale rounds before stopping |
| `--min-improvement` | 0.02 | Minimum relative improvement to reset patience |
| `--refine-top` | 5 | Winners to refine per round |
| `--refine-variants` | 30 | Max variants per winner |
| `--no-loop` | false | Disable refinement (single-pass legacy) |

### Asset universe expansion

Controls which assets are tested during refinement rounds. Round 0 always uses core assets (5) for speed.

| Flag | Default | Description |
|------|---------|-------------|
| `--asset-universe` | `broad` | `core` (5), `broad` (SP500+NDX100 ~550), `full` (all viable warehouse), or preset name |
| `--refine-asset-swaps` | 10 | Max asset swap variants per winner per round |
| `--min-asset-rows` | auto | Min parquet rows for viability (default: train+test sessions) |

### Quality gates

Filter final report to only "investable" strategies. Applied after all rounds complete.

| Flag | Default | Description |
|------|---------|-------------|
| `--min-sharpe` | none | Minimum test-window Sharpe ratio |
| `--max-drawdown` | none | Maximum test-window drawdown (absolute %, e.g. `20` = reject worse than -20%) |

When gates are active, the report shows pass/fail counts. If none pass, unfiltered results shown for reference.

### Convergence summary

When the refinement loop terminates (patience, frontier exhaustion, or max rounds), a diagnostic summary prints:
- Stop reason
- Total evaluated/passed counts and exhausted refinement centers
- Rule and asset distribution among passing candidates
- Score trajectory from first to last round

### Evaluation cache

`reports/autoresearch-eval-cache.jsonl` persists results across runs keyed by parameter signature + date windows. Deterministic grid and repeated seeded candidates are served instantly. Use `--no-cache` to force re-evaluation (e.g., after strategy code changes).

### Audit trail and registry

- Persisted top-10 winners must have train/test audit artifacts under `reports/autoresearch-audits/` with exact evaluated periods, trade ledgers, and equity traces.
- Promoted winners must also be upserted into `reports/autoresearch-strategy-registry.json`, keyed by stable parameter signature, so future implementation work can start from audited candidates rather than only JSONL history.

## Data and candidate constraints

- Backtests use local warehouse parquet data only.
- Loop candidates must execute as `paper-research` only.
- The loop should run with Exa/arXiv candidate seeding via `--seed-web` for new strategy discovery.
- Candidate pool should target at least 100 research candidates by default.

## Output artifacts

- `reports/autoresearch-ledger.jsonl` — append-only log of top-ranked results per run
- `reports/autoresearch-exa-ideas.json` — raw Exa/arXiv seeds (when `--seed-web` is set)
- `reports/autoresearch-top10-interactive-report.html` — branded interactive report
- `reports/autoresearch-audits/` — auditable train/test proof JSONs for persisted top-10 candidates
- `reports/autoresearch-strategy-registry.json` — promoted strategy registry for future real-world implementation work
- `reports/autoresearch-eval-cache.jsonl` — persistent evaluation cache (cross-run)
- Prefer reading these after each production loop before promoting candidates.

## Required behavior for the loop

- Candidate discovery uses arXiv-focused Exa search when `--seed-web` is provided.
- Build net-new candidates from web-seeded hypotheses + deterministic paper-research mutations.
- Run walk-forward train/test windows.
- Score train/test and rank using the loop scoring formula.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DRose-Devs/zonk](https://github.com/DRose-Devs/zonk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
