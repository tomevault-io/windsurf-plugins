---
trigger: always_on
description: This skill's failure mode is a plausible-looking CSV with **wrong values**. Full table:
---

# Agent-with-OpenROAD — Project Guide

AI-driven open-source EDA flow: natural-language spec → GDSII via OpenROAD-flow-scripts
(ORFS), with full signoff (DRC, LVS, RCX), then a **training-ready graph dataset** for GNN
predictors. Implemented as the `r2g-skills` Claude Code skill collection — **four sub-skills**
(`signoff-loop` + `def-graph` from the 2026-07-07 split, see
`docs/superpowers/plans/r2g-skills-split-2026-07-07.md`; `eda-install` added 2026-07-08, see
`docs/superpowers/plans/r2g-skills-bootstrap-2026-07-08.md`; `rtl-acquire` ingested 2026-07-09, see
`docs/superpowers/plans/rtl-acquire-ingestion-2026-07-09.md`):
- **`eda-install`** — detects the machine and installs + verifies the toolchain the others run
  (ORFS + openroad/yosys, iverilog, klayout, magic/netgen, sky130A PDK, torch venv). One command,
  `bootstrap.sh` (detect → plan → install → pin `env.local.sh` → verify); no-sudo conda path by default.
- **`rtl-acquire`** — the RTL corpus supplier, UPSTREAM of the others: discovers/screens/acquires RTL
  at corpus scale (local trees, repo manifests, keyword search) and expands it **synth-only** into
  pre-layout `netlist_graph.pt` graphs with dedup, quality scoring, and publish gating. Owns
  acquire + corpus publish + the one-click **promote** of a synth-proven candidate into a
  signoff-loop full-flow project (`scripts/promote/promote_candidates.py`, 2026-07-10); BORROWS
  env (`_env.sh`), synth (`run_orfs.sh`, `ORFS_STAGES=synth`),
  the graph format (def-graph `netlist_graph.py`), and failure learning (`knowledge.sqlite`,
  runs stamped `flow_scope='synth_only'`; frontend classes land as `synth-frontend-*` events).
- **`signoff-loop`** — drives the flow RTL→GDS with full signoff *and* the self-improvement loop
  (the two memory DBs + `engineer_loop`) that eliminates DRC/LVS violations and closes timing at Fmax.
- **`def-graph`** — converts the clean, signed-off physical design (the ORFS `6_final.odb`/`.def`/
  `.spef` + platform liberty/LEF) into PyTorch-Geometric graph datasets: five graph views (b–f), emitted
  as `HeteroData` by default (2026-07-16; `R2G_GRAPH_KIND=homo` for the legacy flat tensors), the
  shared tech-lib/LEF/DEF parser, and feature (X) / label (Y) extraction — labels are congestion,
  wirelength, per-path timing slack, IR drop, and SPEF-derived RC parasitics (the last a `y[N,6]` node
  label + a separate `rc_edge_*` parasitic edge set, merged 2026-07-07).

**Each skill has ONE heart; everything else is plumbing** — read the two ⭐ sections below:
1. **`signoff-loop` → The Closed Learning Loop** — the two memory DBs (`knowledge.sqlite` = what
   *resulted*, `journal.sqlite` = what was *done*) + `engineer_loop`, the autonomous driver that closes
   the wheel unattended (flow → fix → learn → A/B-promote) and learns repair recipes that transfer
   across designs/platforms.
2. **`def-graph` → The Dataset-Construction Pipeline** — three composable stages (labels → features →
   graphs) keyed to the *same* DEF so X and Y join, whose failure mode is a plausible CSV with silently
   *wrong values*.

This file is *orientation*; the skills document *how* to run/debug/tune. **Don't duplicate skill content
or per-run results here** — when you fix a bug, update the relevant sub-skill under `r2g-skills/`
(`signoff-loop/` for flow/signoff/learning, `def-graph/` for dataset construction), not this file. Prefer
editing existing `scripts/` over adding new ones; use the documented steps, not ad-hoc shell, in production.

## Project Layout

```
r2g-skills/                     # The skill collection — installs THREE Claude Code skills
  install.sh                      # Installs all four sub-skills (symlinks each into .claude/skills/)
  bootstrap.sh                    # Shim → eda-install/bootstrap.sh (documented one-command setup)
  eda-install/                  # SKILL 0 — detect + install + verify the EDA toolchain (no-sudo default)
    SKILL.md                      # detect → plan → install → pin env.local.sh → verify
    bootstrap.sh                  # The orchestrator (--dry-run plans; --yes installs)
    scripts/setup/                # detect_env.sh, write_env_local.sh (+ install_<tier>.sh)
    scripts/flow/                 # _env.sh (byte-identical copy), check_env.sh (comprehensive verifier)
  signoff-loop/                 # SKILL 1 — RTL→GDS flow + signoff + the self-improvement loop
    SKILL.md                      # Workflow, hard rules, env knobs (PLACE_FAST, ROUTE_FAST, …)
    scripts/flow/                 # Stage runners: run_orfs.sh, run_drc/lvs/rcx.sh, fix_signoff.sh, _env.sh
      orfs_hooks/                   # ORFS stage-hook Tcl (POST_GLOBAL_PLACE_TCL, …)
    scripts/extract/              # Tool output → JSON: extract_ppa/drc/lvs/rcx/route + report_io, presynth
    scripts/project/              # init_project, normalize_spec, validate_config
    scripts/reports/              # check_timing, diagnose_signoff_fix, fmax_search, build_*
    scripts/loop/                 # engineer_loop.py — the autonomous campaign driver
    scripts/dashboard/            # render_gds_preview, generate/serve dashboard
    knowledge/                    # The two memory DBs + learn/ingest/A-B Python (self-contained)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ShenShan123/r2g-skills](https://github.com/ShenShan123/r2g-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
