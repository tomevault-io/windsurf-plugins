---
trigger: always_on
description: Maintain a deterministic, local Python tool for writing analysis. Make the smallest complete change that satisfies the user's request. Preserve public CLI behavior, JSON schemas, report determinism, and benchmark provenance unless the task explicitly changes one of those contracts.
---

# Writing Eval Repository Instructions

<task>
Maintain a deterministic, local Python tool for writing analysis. Make the smallest complete change that satisfies the user's request. Preserve public CLI behavior, JSON schemas, report determinism, and benchmark provenance unless the task explicitly changes one of those contracts.
</task>

<instruction_priority>
Apply instructions in this order:

1. The user's stated task and acceptance criteria.
2. The behavioral and data contracts in this file.
3. The documented public interface in `README.md`.
4. Existing tests and implementation details.

Use current code and focused tests to resolve internal details. Historical benchmark planning and status notes are not an active backlog and do not override the current CLI, tests, or user request.
</instruction_priority>

<default_follow_through_policy>
- Default to a reasonable low-risk interpretation and continue without asking routine questions.
- Ask only when missing information changes correctness, data safety, benchmark validity, or an irreversible action.
- Inspect the relevant implementation, tests, and documented interface before editing.
- Complete implementation, caller migration, tests, documentation, and verification in the same task.
- Do not stop at a plan, partial fix, plausible explanation, or passing focused test.
</default_follow_through_policy>

<repository_map>
## Runtime and entry points

- Python 3.11 or newer, managed with `uv`.
- `writing-eval` and `src/writing_eval/cli.py` expose the command-line interface.
- `scripts/run_eval.py` is the corpus evaluation wrapper. `scripts/` holds only this file and `dry_run.sh` (the product-facing corpus path).
- `scripts/dry_run.sh` exercises the bundled end-to-end sample.
- `benchmark/` is the closed internal benchmark harness, including its scripts, frozen `eval-config.json`, and provenance records. See `benchmark/README.md`.

## Core package

- `cli_check.py`, `cli_check_render.py`, `cli_profile.py`, and `cli_eval.py` implement command families. Keep `cli.py` as the thin dispatcher and shared CLI boundary.
- `style_audit*.py` contains rule models, loading, detection, and audit execution. `style_audit_paths.py` holds `BUILTIN_RULES_PATH`, the one default rule-set path every entry point uses. `style_audit_overlay.py` resolves `extends` overlay chains into one merged raw rule list before validation.
- `metrics*.py` and `segmentation.py` contain deterministic text measurements.
- `assessment*.py` builds and renders single-document assessments.
- `profile*.py` and `profiles.py` manage local style profiles.
- `report*.py` builds deterministic corpus report data and Markdown.
- `comparison*.py` implements system comparison, noise floors, and the decision gate.
- `generation*.py` implements generation input, prompting, and run orchestration; `benchmark/codex_runner.py` holds the Codex provider subprocess.
- Small modules such as `style_audit.py`, `metrics.py`, `assessment.py`, `comparison.py`, `profiles.py`, and `report.py` are public facades. Keep logic in the focused implementation modules instead of growing the facades.

## Data, rules, and records

- `tests/` contains behavior-focused pytest coverage.
- `src/writing_eval/rules/style-audit.yaml` is the single builtin rule set, shipped inside the installed package. Users add, override, or disable rules with an `extends` overlay file passed to `--rules` instead of forking this file.
- `rules/anti-ai.yaml` is the optional repository-level overlay (`extends: builtin`) that appends four AI-tell rules and widens six builtin ones for opt-in single-draft checks. It never ships inside the package and is never a default; corpus evaluation and benchmark runs keep the builtin rules.
- `tests/fixtures/` contains small, nonconfidential, deterministic test inputs.
- `benchmark/eval-config.json` holds three frozen-decoding fields (model, reasoning_effort, system_prompt) read by `benchmark/generate_runs.py`; every other field in it is unread.
- `benchmark/THRESHOLDS.md` and `benchmark/SAMPLES.md` are sealed historical provenance records, matching `benchmark/README.md`. They record how the closed experiment was run and do not define live tool behavior.
- `docs/profile-size-study.md` records the measurement behind the corpus-size recommendation in `README.md`. Unlike the benchmark records, it is live documentation: update it when the measurement changes.
- `docs/profile_size_study.py` reproduces all three experiments in that document from a built profile's `references.jsonl`. Every published table must reproduce from `--references` alone. Changing the script's defaults, sampling, or seed invalidates the published tables, so re-verify all three before committing such a change.
- `docs/banner.png` is the README banner image.
- Style-profile corpus sizing comes from `docs/profile-size-study.md`. The sample-count and word-count language in `benchmark/SAMPLES.md` describes the closed benchmark's reference corpus and does not govern style-profile sizing.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [majesticlabs-dev/writing-eval](https://github.com/majesticlabs-dev/writing-eval) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
