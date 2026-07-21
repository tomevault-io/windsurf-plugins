---
trigger: always_on
description: `yao-meta-skill` is the source repository for the Yao Meta Skill / Skill OS. Keep the root `SKILL.md` lean and route depth through the existing project layers:
---

# Agent Instructions

## Project Map

`yao-meta-skill` is the source repository for the Yao Meta Skill / Skill OS. Keep the root `SKILL.md` lean and route depth through the existing project layers:

- `SKILL.md`: public trigger surface, compact workflow, and Skill OS gates.
- `references/`: stable method doctrine and operating guidance.
- `scripts/`: executable logic, report generators, compilers, gates, and CLI entrypoints.
- `evals/`: trigger, output, packaging, semantic, and regression fixtures.
- `skill-ir/`: platform-neutral skill contract and examples.
- `agents/interface.yaml`: portable runtime and trust metadata.
- `security/`: script, dependency, network, permission, and trust policies.
- `registry/`: package metadata, installability evidence, and compatibility metadata.
- `skill_atlas/`: portfolio catalog, route overlap, ownership, stale-skill, and dependency evidence.
- `reports/`: generated review, trust, conformance, registry, overview, and release evidence.
- `docs/`: durable public docs such as migration notes and localized READMEs.

## Verification

Use `make ci-test` as the default full verification command before calling a change done. For focused work, run the smallest relevant target first, then finish with `make ci-test` when generated artifacts, packaging, registry, trust, or report UI changed.

Common focused checks:

- CLI changes: `python3 tests/verify_yao_cli.py`
- Operator UX changes: `python3 tests/verify_operator_ux.py`
- Skill overview report changes: `python3 tests/verify_skill_overview.py`
- Review Studio changes: `python3 tests/verify_review_studio.py`
- Trust or script inventory changes: `python3 tests/verify_trust_check.py`
- Packaging or registry changes: `python3 tests/verify_package_verification.py && python3 tests/verify_registry_audit.py`

After source changes that affect scripts, package contents, trust evidence, Review Studio, registry metadata, or generated reports, refresh the release evidence before final sign-off:

```bash
GENERATED_AT="${GENERATED_AT:-$(date +%F)}"
python3 scripts/run_output_execution.py --runner-command '["python3","scripts/local_output_eval_runner.py"]'
python3 scripts/compile_skill.py . --generated-at "$GENERATED_AT"
python3 scripts/cross_packager.py . --platform openai --platform claude --platform generic --platform vscode --expectations evals/packaging_expectations.json --output-dir dist --zip
python3 scripts/simulate_install.py . --package-dir dist --install-root dist/install-simulation --output-json reports/install_simulation.json --output-md reports/install_simulation.md --generated-at "$GENERATED_AT"
python3 scripts/trust_check.py . --output-json reports/security_trust_report.json --output-md reports/security_trust_report.md
python3 scripts/registry_audit.py . --generated-at "$GENERATED_AT"
python3 scripts/verify_package.py . --package-dir dist --expectations evals/packaging_expectations.json --registry-json reports/registry_audit.json --output-json reports/package_verification.json --output-md reports/package_verification.md --require-zip --generated-at "$GENERATED_AT"
python3 scripts/registry_audit.py . --generated-at "$GENERATED_AT"
python3 scripts/upgrade_check.py . --previous-package-json registry/examples/yao-meta-skill-1.0.0.json --current-package-json reports/registry_audit.json --output-json reports/upgrade_check.json --output-md reports/upgrade_check.md --generated-at "$GENERATED_AT"
python3 scripts/render_adoption_drift_report.py . --generated-at "$GENERATED_AT"
python3 scripts/render_architecture_maintainability.py . --generated-at "$GENERATED_AT"
python3 scripts/python_compat_check.py . --generated-at "$GENERATED_AT"
python3 scripts/probe_runtime_permissions.py . --package-dir dist
python3 scripts/render_review_waivers.py . --generated-at "$GENERATED_AT"
python3 scripts/render_review_annotations.py .
python3 scripts/build_skill_atlas.py --workspace-root . --output-dir skill_atlas --report-html reports/skill_atlas.html --report-json reports/skill_atlas.json --today "$GENERATED_AT"
python3 scripts/render_world_class_evidence_plan.py . --generated-at "$GENERATED_AT"
python3 scripts/render_world_class_evidence_ledger.py . --generated-at "$GENERATED_AT"
python3 scripts/render_world_class_evidence_intake.py . --generated-at "$GENERATED_AT"
python3 scripts/render_world_class_submission_review.py . --generated-at "$GENERATED_AT"
python3 scripts/render_world_class_operator_runbook.py . --generated-at "$GENERATED_AT"
python3 scripts/render_world_class_claim_guard.py . --generated-at "$GENERATED_AT"
python3 scripts/render_daily_skillops_report.py . --generated-at "$GENERATED_AT"
python3 scripts/render_weekly_curator_report.py . --generated-at "$GENERATED_AT"
python3 scripts/render_skill_os2_audit.py . --generated-at "$GENERATED_AT"
python3 scripts/render_skill_os2_coverage.py . --generated-at "$GENERATED_AT"
python3 scripts/render_context_reports.py --generated-at "$GENERATED_AT"
python3 scripts/render_benchmark_reproducibility.py . --generated-at "$GENERATED_AT"
python3 scripts/render_skill_overview.py .
python3 scripts/render_skill_interpretation.py .
python3 scripts/render_review_viewer.py .

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yaojingang/yao-meta-skill](https://github.com/yaojingang/yao-meta-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
