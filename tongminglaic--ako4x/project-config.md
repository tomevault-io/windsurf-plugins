---
trigger: always_on
description: Template repository for spawning GPU kernel optimization environments.
---

# AKO4X — Developer Guide

Template repository for spawning GPU kernel optimization environments.
**Not** an optimization environment — use `spawn.py` to create one.
User-facing docs: [README.md](README.md) and [docs/](docs/).

## Architecture

Four layers:

1. **`spawn.py`** — CLI. Creates child environments from templates + dataset + scripts. Also copies `templates/skills/` to `child/.claude/skills/` (Claude Code progressive-disclosure discovery) and `scripts/CLAUDE.md` to `child/scripts/CLAUDE.md`.
2. **`templates/`** — canonical sources copied / rendered into each child.
   - **`task.md`** — frozen task identity + Workflow, with `{{PLACEHOLDER}}` substitutions.
   - **`retrospective.md`** — phase-2 closed-loop prompt.
   - **`agent/`** — `<agent>.json` (per-agent config, currently `claude.json`; selected via `spawn.py --agent`) + `lessons-convention.md` + `hooks/` + `commands/`.
   - **`iterations.md`** — iteration-log template.
   - **`benchmark/evaluation.toml`** — benchmark-bound bench defaults + per-`op_type` tolerance overrides. `templates/benchmark/` is the active benchmark's template dir; its name is the stable slot `spawn.py` reads via the `BENCHMARK_DIR` constant.
   - **`skills/<name>/{SKILL.md, <doc>.md}`** — 9 SKILLs (bench, benchmark, profiler-ncu, sanitizer, triton, cuda, cute-dsl, tilelang, cpp). `bench` carries generic noise-aware methodology; `benchmark` carries the active benchmark's schema (config.toml, status enum, scoring, baseline rule, fresh-inputs contract; default content is flashinfer-bench). The bench/benchmark split's load-bearing role is master FROZEN-scope enforcement — master reads `benchmark` SKILL's "Frozen for bench comparability" section at step 7.

   **Single-active-benchmark assumption.** The repo assumes one active benchmark at a time (bench-runner + task set, both swap together) — no `benchmarks/` plural-container, no runtime selector flag. Multi-benchmark *coexistence* (several behind a per-spawn selector) is a different, larger thing — deferred under YAGNI until a second benchmark is actually needed.

   **Benchmark decoupling.** The benchmark is decoupled behind one seam: `scripts/benchmark_adapter.py` is the sole `flashinfer_bench` importer, and the generic skills point at the stable `benchmark` slot (not a benchmark-specific name), so a swap does NOT touch the runners, the generic DSL skills, or `bench_utils.py`'s scoring math. Switching benchmarks = rewrite `scripts/benchmark_adapter.py` (its plain-data public functions — `run` / `pack` / `solution_meta` / `list_workloads` / `profile` / `list_ncu_options` / `sanitize` / `cheat_check`, with only `str` / `list` / `dict` crossing the seam — plus the Modal-image and dataset-env constants) + the `benchmark` skill's content + `templates/benchmark/evaluation.toml` + the `flashinfer-bench` dependency in `pyproject.toml`. `scripts/bench_utils.py` keeps the frozen `compute_score` / `load_baseline` / `save_baseline` math, which operates on the adapter's normalized result dict and is benchmark-agnostic (no benchmark types cross into it). Full agent-followable procedure: **`docs/porting.md`**.
3. **`scripts/`** — Most files copied into children (canonical list lives in `spawn.py`'s explicit copy allowlist). Sub-visible: `CLAUDE.md` (shared-runtime-core contract for closed-loop), `benchmark_adapter.py` (the sole `flashinfer_bench` importer — the benchmark seam; everything else reaches the benchmark through it), `bench_utils.py` (shared core, frozen-for-comparability segments around `compute_score` / `load_baseline` / `save_baseline`), `run_local.py` / `run_modal.py` (runners), `run_{local,modal}_{profile,sanitize}.py` (NCU + sanitizer wrappers), `pack_solution.py`, `diff_trajectory.py`. **Parent-only** (NOT copied to children): `cheat_check_modal.py` (modal-only correctness audit, invoked as `modal run …/cheat_check_modal.py`) and `backfill_parent_txt.py` (one-shot variant-lineage filler over `reference/`).
4. **Closed-loop scaffolding (`master/`, opt-in via `master/MASTER.md`)** —
   - **`master/master.py`** — thin IO layer: 8 functions (`init_campaign`, `read_campaign_mode`, `spawn_child`, `run_sub_phase1`, `send_retrospective_prompt`, `archive_variant`, `archive_failed`, `append_ledger`), no decision logic. Importable as `import master` from repo root via `master/__init__.py` re-export.
   - **`master/MASTER.md`** — master CC system prompt + 10-step round loop with **two modes**: Mode 2 default = no harness modification, sub does phase-1 kernel optimization only; Mode 3 opt-in = harness co-evolution, sub additionally writes `PROPOSALS.md` in phase-2 and master evidence-gates and applies accepted edits.
   - **`master/harness-ledger.md`** — append-only timeline of harness edits + Mode-2 round-summary lines.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TongmingLAIC/AKO4X](https://github.com/TongmingLAIC/AKO4X) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
