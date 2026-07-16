---
trigger: always_on
description: - **Hard rule — Qsys edits go through Tcl, never raw XML.** Any change to a Platform Designer system (`*.qsys`, `*.sopcinfo`, `synthesis/`, `*.ipx`) must be made by a `qsys-script` or `qsys-edit` Tcl recipe under `script/` and applied through the Tcl regenerator. Direct hand-edits of the qsys XML, the generated synthesis tree, or the IPX library file are forbidden, even when "just one line." Reason: qsys files carry implicit address-map, mm_interconnect, and reset/clock fanout state that is reco
---

# Agent Notes

- **Hard rule — Qsys edits go through Tcl, never raw XML.** Any change to a Platform Designer system (`*.qsys`, `*.sopcinfo`, `synthesis/`, `*.ipx`) must be made by a `qsys-script` or `qsys-edit` Tcl recipe under `script/` and applied through the Tcl regenerator. Direct hand-edits of the qsys XML, the generated synthesis tree, or the IPX library file are forbidden, even when "just one line." Reason: qsys files carry implicit address-map, mm_interconnect, and reset/clock fanout state that is recomputed at generate time; a hand edit silently desyncs against the regenerator script, and the next regenerate run loses the change. Each regenerator script must `chmod -R a-w` the generated `*.qsys`, `*.sopcinfo`, and `synthesis/` tree on success so the contract is enforced by the filesystem, not just by discipline.
- **FEB SciFi v4 golden Qsys reference follow-up.** The 2026-06-05 real-MuTRiG full-channel reference is captured in `firmware_builds/systems/260526-feb-bug012/scripts/report/real_mutrig_alive_matrix_20260604/post_rewire_load_test/POST_REWIRE_LOAD_TEST_20260604.md` and commit `9af7c260`. Treat the working board state as the golden FEB reference for the next Qsys task: fix the remaining Qsys wiring/regeneration drift and convert the FEB Platform Designer system into a deterministic Tcl generator that recreates the golden reference, including 1:1 injector pin routing, 32-bit histogram counters, and the ASIC2-last MuTRiG configuration procedure. Validate with Platform Designer GUI validation, `qsys-generate`, timing, firmware load, and the same Type0/Type1 rate plus header-sync delay plots before accepting the Tcl generator as source of truth.
- Never modify generated RTL under any `functional/` output directory in-place. If a change is needed, create a copy and ask for approval before wiring it in.
- Semantics-preserving edits for tool/simulator compatibility are OK; functional/behavior changes require explicit approval.
- Shared System Console toolkit sources live under `toolkits/`.
- The FE SciFi toolkit source of truth is `toolkits/fe_scifi/`.
- `online_dpv2` consumes the FE SciFi toolkit through the compatibility path `/home/yifeng/packages/online_dpv2/online/mu3e-ip-cores/toolkits`, which must resolve back to this repo.
- `/home/yifeng/packages/online_dpv2/online/mu3e-ip-cores` is only a compatibility symlink into the deprecated snapshot area. Do not add or update toolkit sources under its `toolkits/` tree.
- **SWB+FEB debug + bring-up tools** (`sc_tool`, `rc_tool`, `dma_tool`, `run_tool`, `test_scifi_sc_hub`, etc.) now live at `tools/run_script/`. Moved from `online_dpv2/online/switching_pc/tools/` + `online_sc/online/switching_pc/tools/` on 2026-05-11. The old paths now carry `MOVED.md` redirect notes and are frozen — do not edit there.
- **FEB SciFi builds** live at `firmware_builds/systems/v3_pretest-260511/` (current head) and `firmware_builds/systems/system_20260427_testplanphase5/` (Apr 27 reference floor; the build that produced the good FEB hit-lifetime profile, used as the IP-version floor for v3 qsys per `memory/project_v3_qsys_floor.md`). The legacy `online_dpv2/online/fe_board/fe_scifi/` location is deprecated for new compiles.
- **SWB builds** live at `firmware_builds/systems/swb/rdma_pretest-260511/` (current head). The legacy `online_sc/online/switching_pc/a10_board/` location is deprecated for new compiles.
- For RTL modeling tasks, use the `modeling-rtl` workflow: start from upstream slides/spec/source RTL and analytical truth, then make TLM, RTL simulation, and on-board evidence converge to that abstraction.
- `firmware_builds/systems/system_20260427_testplanphase5/model/` is the durable three-tier model tree (`analytical/`, `tlm/`, `on_board/`, and phase-specific model evidence). `firmware_builds/systems/system_20260427_testplanphase5/script/`, `reports/`, and `signaltap/` are the live board-test locations; canonical test plans live under `firmware_builds/doc/`.
- Treat the Mu3e slides and reviewed upstream source artifacts as truth. If TLM, RTL simulation, and board evidence disagree in an abstracted DISLIN plot, debug the lower level first instead of tuning the high-level model to a bad lower-level observation.
- Phase 4 closure is gated by `firmware_builds/doc/phase4/TEST_PLAN_BASIC.md`: the BASIC-like bucket must include TLM, RTL simulation, on-board, and cross-layer DISLIN evidence for its directed cases before Phase 4 is called closed.

## Phase 4 emulator + histogram verification workflow (TLM -> RTL SIM -> board)

Authoritative catalog: [`firmware_builds/doc/phase4/TEST_PLAN_BASIC.md`](firmware_builds/doc/phase4/TEST_PLAN_BASIC.md).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yifeng-ethz/mu3e-ip-cores](https://github.com/yifeng-ethz/mu3e-ip-cores) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
