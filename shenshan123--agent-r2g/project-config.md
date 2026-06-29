---
trigger: always_on
description: AI-driven open-source EDA flow: natural-language spec → GDSII via OpenROAD-flow-scripts
---

# Agent-with-OpenROAD — Project Guide

AI-driven open-source EDA flow: natural-language spec → GDSII via OpenROAD-flow-scripts
(ORFS), with full signoff (DRC, LVS, RCX). Implemented as the `r2g-rtl2gds` Claude Code skill.

**Two things make this project what it is** (everything else is plumbing):
1. **Two memory databases** (`knowledge.sqlite` = what *resulted*, `journal.sqlite` = what was
   *done*) record every run and learn repair recipes that transfer across designs/platforms.
2. **`engineer_loop`** — the autonomous driver that closes the loop unattended (flow → fix →
   learn → A/B-promote) and keeps the skill self-evolving.

See "The Closed Learning Loop" below — it is the heart of the repo. This file is *orientation*;
the skill documents *how* to run/debug/tune. **Don't duplicate skill content or per-run results
here** — when you fix a bug, update `r2g-rtl2gds/` (the skill), not this file. Prefer editing
existing `scripts/` over adding new ones; use the documented steps, not ad-hoc shell, in production.

## Project Layout

```
r2g-rtl2gds/                  # The skill (everything to run a flow lives here)
  SKILL.md                      # Workflow, hard rules, env knobs (PLACE_FAST, ROUTE_FAST, …)
  scripts/flow/                 # Stage runners: run_orfs.sh, run_drc/lvs/rcx.sh, fix_signoff.sh
    orfs_hooks/                   # ORFS stage-hook Tcl (POST_GLOBAL_PLACE_TCL, …)
  scripts/extract/              # Tool output → JSON: extract_ppa/drc/lvs/rcx/route; techlib/, labels/, features/
  scripts/project/              # init_project, normalize_spec, validate_config
  scripts/reports/              # check_timing, diagnose_signoff_fix, suggest_config, build_*
  scripts/loop/                 # engineer_loop.py — the autonomous campaign driver
  scripts/dashboard/            # render_gds_preview, generate/serve dashboard
  knowledge/                    # The two memory DBs + learn/ingest/A-B Python (self-contained)
  references/                   # Detailed docs (see "Where to find X")
  assets/  tests/               # Templates + bundled platform extras; pytest suite
tools/                          # Repo-level operator tooling + installers
design_cases/                   # All design runs (gitignored); _batch/, _dashboard/
```

## Skill Deployment (must be a symlink, not a copy)

Claude Code loads the skill from `.claude/skills/r2g-rtl2gds/` (gitignored), **not** the canonical
`r2g-rtl2gds/` tree. Deploy with `bash r2g-rtl2gds/install.sh --project . --link` so the path is a
**symlink**. A plain `cp` install silently goes stale — the harness then loads an old `SKILL.md`
while the canonical skill evolves. If a session's loaded skill disagrees with `r2g-rtl2gds/SKILL.md`,
re-run with `--link --force`. (Root cause of the 2026-06-08 stale-skill defect.)

## Toolchain (autodetected by the skill)

`scripts/flow/_env.sh` autodetects ORFS + tool paths — nothing to source manually. Override via
`$R2G_ENV_FILE`, `references/env.local.sh`, or by exporting `ORFS_ROOT`/`OPENROAD_EXE`/`YOSYS_EXE`/
`KLAYOUT_CMD`/… **Required:** python3 (3.10+), yosys, openroad, ORFS checkout. **Optional:**
iverilog/vvp, verilator, klayout, magic, netgen-lvs, opensta, sky130A PDK. Verify with
`scripts/flow/check_env.sh`.

**This machine:** signoff tools (iverilog/vvp, magic, netgen) live in `~/miniconda3/envs/eda`; the
sky130A PDK is staged at `/proj/workarea/user5/sky130_pdk/share/pdk/sky130A`; all pinned in
`references/env.local.sh` and green in `check_env.sh` (enables real sky130 Magic DRC + Netgen LVS).
Install recipe in `README.md`. **Never install large packages into `$HOME` (full) — use `/proj`.**
Platforms in this checkout: `nangate45` (default), `sky130hd`, `sky130hs`, `asap7`, `gf180`,
`ihp-sg13g2`. The nangate45 LVS rule is bundled (`tools/install_nangate45_lvs.sh`).

## Hard Rules (skill-level)

- **Never run two configs with the same `DESIGN_NAME` + `FLOW_VARIANT` concurrently.** `run_orfs.sh`
  derives `FLOW_VARIANT` from the project-dir basename — keep names unique within a `DESIGN_NAME`.
- **Never set `PLACE_DENSITY_LB_ADDON` below 0.10.** Placer divergence is irrecoverable.
- **For >100K-cell designs, never run multiple LVS jobs concurrently** (3-5GB RAM each → 2-3× wall time).
- **Parallel ORFS:** when running flows concurrently, cap per-flow threads with `NUM_CORES` so
  `flows × NUM_CORES ≈ cores` — the default grabs `nproc` (96) per flow, so N flows oversubscribe N×
  and thrash. `run_orfs.sh` wraps each stage in `setsid timeout`, so killing a driver orphans the
  make/openroad tree — `kill -9 -<pgid>` the process group, not just the python.
- **Escalate to the user before attempting CDC, multi-clock, DFT, or signoff-quality closure.**
  Single-clock flows incl. macro designs (`fakeram45`) are supported; the rest is out of scope.
- **Don't skip a failed stage** — diagnose first via `references/failure-patterns.md`. The strict
  flow order (spec → … → RCX → reports) lives in `SKILL.md`; **ingest after every flow** (clean,
  failed, or partial) so the learning loop sees it.

## The Closed Learning Loop  ⭐ (memory databases + engineer_loop)

The skill *learns from every run*. **Detail — schema, CLI, the full numbered invariants — lives in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ShenShan123/agent-r2g](https://github.com/ShenShan123/agent-r2g) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
