---
trigger: always_on
description: >-
---


# Erie Verilog Generator

Use this skill for Verilog-2001 RTL generation and existing-RTL analysis/refinement/verify-repair backed by the bundled `runtime/verilog_generator` Python workflow. Keep generated design RTL artifacts as Verilog `.v` files and use the stable facade in `integration/verilog_adapter.py`.

For new RTL generation, expose three user-facing workflow shells:

- `regular`: the standard staged generation path
- `deep_review`: one extra structured review stage before final RTL emission
- `agentic_repair`: the existing `verify_existing_verilog(...)` evidence-driven repair flow for existing RTL

These are workflow modes, not alternate correctness gates. Do not treat free-form model self-assessment text such as `[DESIGN IS CORRECT]` as authoritative validation evidence.

The same skill also exposes two local helper scripts for independent static lint and testbench scaffold generation without widening the skill beyond Verilog-2001:

- `scripts/verilog_lint.py` for independent static lint
- `scripts/tb_generator.py` for a self-checking Verilog testbench scaffold

These helper tools are optional workflow steps. They are part of the skill execution flow when the request benefits from them, but they are not mandatory entry gates.

For existing RTL assets, the same runtime now exposes:

- `analyze_existing_verilog(...)` to emit `rtl_analysis.json`, `project_analysis.json`, and `design_explanation.md`
- `refine_existing_verilog(...)` to emit `rtl_transform_plan.json` plus controlled helper artifacts such as a scaffold testbench, partition wrapper skeleton, merge-assist bundle, or optimize-assist plan bundle
- `compare_verilog_semantics(...)` to emit `equivalence.json`, `qor_report.json`, and `transform_validation.json`
- `verify_existing_verilog(...)` to emit `project_analysis.json`, `verification_plan.json`, `tb_contract.json`, `log_diagnosis.json`, `patch_candidate.json`, `verification_result.json`, and `loop_state.json`
  The same verify-repair flow also emits `simulation_slice.json`, `timing_diagnostic.json`, `expected_trace.md`, `waveform_diff.json`, `testcase_matrix.json`, `run_summary.json`, `synth_readiness.json`, and `terminal_status.json` so the verification closure is explicit even when the run stays in static mode.
  When RTL repair is applicable, the same flow also emits `rtl_patch_plan.json`, `rtl_patch_diff.txt`, `rtl_intervention.json`, `post_apply_validation.json`, and `post_apply_equivalence.json`.

## Dependency Preflight

On first use in a Codex installation, and before any remote/Vivado/Vitis-related workflow, run the dependency check from this skill root:

```powershell
python .\scripts\manage_skill_dependencies.py check --settings .\config\defaults.json
```

If required dependencies are missing, run `prompt` and ask the user whether to install each missing dependency before continuing:

```powershell
python .\scripts\manage_skill_dependencies.py prompt --settings .\config\defaults.json
```

Install only after the user confirms, then run `adapt` and tell the user to restart Codex so newly installed skills are discovered. Do not install `fpga-agent-skills` during normal preflight. If FPGA developer tooling is missing, prefer `vivado-developer`, `vitis-developer`, or `pds-developer`; FPGA-Agent-Skills is manual fallback only. If the user declines required dependency installation, continue only with local static Verilog generation/validation and block remote SSH, Vivado, Vitis, execute, and implement readiness paths. If recommended dependencies are missing, ask the user whether to install or skip them; use `skip <dependency-id>` only for a user-declined recommended dependency.

## FPGA Developer Routing

Before FPGA simulation, synthesis, implementation, constraints, debug, project creation, or other vendor tool work, inspect the developer routing state:

```powershell
python .\scripts\manage_skill_dependencies.py fpga-route --settings .\config\defaults.json
```

If both AMD-Xilinx and PangoMicro developer skills are available and no current selection exists, ask the user which vendor to use for this workflow. Persist only the user-confirmed vendor choice:

```powershell
python .\scripts\manage_skill_dependencies.py select-fpga-vendor --settings .\config\defaults.json amd_xilinx
python .\scripts\manage_skill_dependencies.py select-fpga-vendor --settings .\config\defaults.json pangomicro
```

Developer routing preference is: `vivado-developer`, then `vitis-developer`, for AMD-Xilinx work; `pds-developer` for PangoMicro work. When any of these developer skills is installed, do not install the FPGA-Agent-Skills Vivado/Vitis group. If no developer skill is installed, treat FPGA-Agent-Skills as a manual fallback that requires explicit user direction and the `--allow-fpga-agent-fallback` install flag.

## Workflow

1. Confirm the design intent before generation: module name, ports, clock/reset, behavior, pipeline expectation, interface family, and verification cases.
2. Choose the generation shell explicitly when it matters:
   - `regular` for the default staged pipeline
   - `deep_review` when one extra structured self-review stage would improve prompt completeness or reviewability
   - `agentic_repair` only when starting from existing RTL and needing the verify-repair loop
3. Use the staged pipeline:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Eriemon/verilog-generator](https://github.com/Eriemon/verilog-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
