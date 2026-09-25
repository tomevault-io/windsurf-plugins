---
trigger: always_on
description: ├── server.py           # FastAPI app, all routes, diff logic
---

# Visualizer Development Guide

## Architecture

```
sforge/visualizer/
├── server.py           # FastAPI app, all routes, diff logic
├── scanner.py          # Disk scanner: reads logs/runs/ → Run/Submission models
├── models.py           # Dataclasses: Run, Submission, TestResult
├── markdown.py         # Markdown → HTML renderer (Jinja2 filter)
├── __init__.py
├── __main__.py         # CLI entry: `python -m sforge.visualizer`
├── AGENTS.MD           # This file: dev guide and operator runbook
├── parsers/
│   ├── agent_output.py # Claude Code JSONL → Trajectory/Exchange
│   ├── codex_output.py # Codex plain text → Trajectory
│   └── test_output.py  # Raw test output → per-test blocks
└── templates/
    ├── base.html       # Shell: fonts (Inter/JetBrains Mono), Tailwind, HTMX, CSS
    ├── index.html      # Home: grouped run table with compare checkboxes
    ├── task.html       # Per-task run list
    ├── run_detail.html # Run detail: header + ECharts timeline + submissions + diagnostics
    ├── run_overview.html  # Run overview: all tasks in a run
    ├── compare.html    # Per-task compare: overlaid timelines for selected runs on one task
    ├── compare_runs.html  # Cross-run dashboard: 3-col grid of mini-charts across all shared tasks
    ├── submission_detail.html  # Test results table
    ├── trajectory.html # Full agent conversation + minimap
    ├── _macros.html    # Badge/chip macros (pass_rate_badge, score_badge, etc.)
    ├── _turn.html      # Single exchange rendering (recursive for subagents)
    └── _judger_block.html  # HTMX partial: test output block
```

## Key Design Decisions

### Data Flow
- `scanner.py` re-reads disk on every request (no DB). Docker container status is cached for 5s via a single `docker ps -a` call (replaces per-task `docker inspect`).
- Submission ordering: `evolve_state.json` timestamps > file mtime > seq number.
- Score task detection: `task JSON parser ∈ {score_sum, structured_json}` OR `score_direction` present OR `game_mode`.
- For finalized runs, `final_result.json` is primary; if `best_score=None`, recompute from submissions.
- Agent/model detection for in-progress runs: `_peek_agent()` matches both old format (`Running agent: claude`) and new supervisor format (`Agent supervisor attempt N: claude -p ...`); `_peek_model()` scans multiple lines of `agent_output.txt` for the JSON init event (not just the first line).

### Evolution Timeline (ECharts)
- Step line (`step: "end"`) — score only changes at submissions.
- Milestones = each new-best submission (respects `score_direction` for minimize tasks).
- Agent submissions rendered as circles, auto-eval as triangles (shape differentiation).
- When smoothing is active, step-best line fades (thin + low opacity) and the smooth line becomes the primary visual (thick + solid). Matches W&B convention.
- Pin markers with 2-line labels: `agent-X  score` + analysis summary (20 chars).
- Auto log scale when data spans > 3 orders of magnitude.
- Auto zoom Y-axis to non-zero data range for score tasks.
- Score overflow sentinels (>=1e100, inf, nan) are treated as null in `mv()` to prevent axis squashing.
- `safe_num` Jinja filter ensures Python inf/nan → JS `null` (not literal `inf` which is a JS ReferenceError).
- Tooltip: `appendToBody: true` prevents clipping by chart container.
- Click milestone → detail panel with diff + analysis.

### Smoothing (EMA / SMA)
- Dropdown selector with three modes: Off, EMA (default α=0.6), SMA (default window=30).
- Popover UI: click "Smooth: EMA α=0.60" button to expand slider controls.
- Smoothing is applied **only to milestone points** (kept/new-best), not raw/discarded submissions. This produces a monotonic-ish upper-envelope smooth.
- EMA uses W&B standard formula: `last = α·last + (1-α)·x`, `debias = 1 - α^n`, `smoothed = last / debias`. `last` initialized to 0 (not first value).
- SMA: simple sliding window average over milestone values.
- Consistent across all three chart pages (run_detail, compare, compare_runs).

### Run Overview Lazy Charts
- `/run/{run_id}` lists all tasks in the run as cards but does NOT inline submissions; each mini-chart fetches its own data on demand from `/run/{run_id}/{task}/chart-data` when scrolled into view (`IntersectionObserver`, 600px rootMargin).
- Before this, the page inlined every submission for all tasks; a 49-task / 7103-submission run took >20s server-side. With lazy loading the first paint is sub-second and only visible charts pay the I/O cost.
- ECharts instance is created lazily inside `loadChart`; the loading placeholder div is cleared (`innerHTML = ""`) before `echarts.init`. The page-level `refreshAll` / resize handlers must null-check `c.chart` since unloaded charts have `chart: null`.

### Cross-Run Comparison Dashboard
- Entry: index page checkboxes on **run groups** → "Compare groups (N)" button → `/compare-runs?models=<group>`. The `models=` query name is kept for URL compatibility; the values are run-group prefixes, not API model names.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ByteDance-Seed/EdgeBench](https://github.com/ByteDance-Seed/EdgeBench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
