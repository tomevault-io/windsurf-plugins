---
trigger: always_on
description: - Core entrypoint: `quixi_math_datagen.py` orchestrates dataset builds and samples, instantiating generator classes and handling `--generators` filtering. Mixed-number ops include a random wrapper; factors/GCF/LCM, conversions/comparisons, and order-of-operations generators are wired in.
---

# Repository Guidelines

## Project Structure & Module Organization
- Core entrypoint: `quixi_math_datagen.py` orchestrates dataset builds and samples, instantiating generator classes and handling `--generators` filtering. Mixed-number ops include a random wrapper; factors/GCF/LCM, conversions/comparisons, and order-of-operations generators are wired in.
- Base contract: `base_generator.py` defines `ProblemGenerator.generate()` with required keys (`problem_id`, `operation`, `problem`, `steps`, `final_answer`); `steps` are pipe-delimited strings built with `helpers.step()` and `steps[-1]` must be exactly `Z|<final_answer>`. The pipeline stamps `grade_level`/`difficulty` from `curriculum.py` after `generate()` returns (a generator may emit either key itself to override).
- Generators: `generators/` holds one file per skill (e.g., `multi_digit_addition_generator.py`, `long_division_generator.py`). Add new classes there and to `ALL_GENERATORS`.
- **CRITICAL:** Every new generator class MUST be registered in THREE places:
  1. Add an import statement at the top of `quixi_math_datagen.py` (e.g., `from generators.my_new_generator import MyNewGenerator`)
  2. Add an instance to the `ALL_GENERATORS` list (e.g., `MyNewGenerator()`)
  3. Add a `curriculum.CURRICULUM` entry for the class (grade_level + difficulty) — enforced by `tests/test_datagen_pipeline.py`
  Generators not in `ALL_GENERATORS` will NOT appear in `--sample` output or dataset generation!
- After adding or changing op-codes, regenerate the legend: `uv run python tools/gen_opcode_legend.py` (check freshness with `--check`). The vocabulary is descriptive and organic — new op-codes are fine, but one op-code must keep one field meaning. Reuse an existing code only when the field semantics match.
- Tests: `tests/` mirrors generator names (`test_long_division_generator.py`, etc.) using `unittest`. Keep new tests co-located with matching generator names.
- Catalog: `PROBLEM_TYPES.md` is the authoritative generated catalog and count of problem types. Regenerate it after adding or materially changing a generator.
- Artifacts: JSONL datasets write to repo root unless you pass `-o`. Avoid committing large generated files.

## Build, Test, and Development Commands
- **Virtual environment:** Prefer `uv run python ...` for commands so the project environment is selected explicitly. If not using `uv run`, activate the venv first with `source .venv/bin/activate`.
- Sample run: `uv run python quixi_math_datagen.py --sample` (add `--generators ClassA,ClassB` to limit; add `-s` to fix seed).
- Full dataset: `uv run python quixi_math_datagen.py -n 50000 -o quixi_math_50000.jsonl` (optionally add `--generators ...` and `-s`).
- Builds sample equally per skill (class); override with `--weights "ClassA=2.5,ClassB=0.5"` or a JSON file. Exact `(operation, problem)` repeats are skipped unless `--allow-duplicates`; a per-generator stats table prints at the end.
- Default dataset filename when `-o` omitted: `quixi_math_<n>.jsonl`.
- Tests (all): `uv run python -m unittest discover tests` (or `uv run pytest tests` with the dev group installed).
- Tests (focused): `uv run python -m unittest tests.test_quadratic_generator`.
- Op-code legend: `uv run python tools/gen_opcode_legend.py` regenerates `OPCODES.md`; `--check` verifies freshness.
- Problem catalog: `uv run python tools/gen_problem_types.py` regenerates the user-facing `PROBLEM_TYPES.md` (one entry per generator with a worked example); `--check` verifies freshness. Regenerate after adding or changing a generator.

## Generator Design Principles
- Every arithmetic action should be explicit. If a human would write it in the margin, emit a step for it.
- Steps should be human-legible: show alignment, carries/borrows, trial candidates, rejected paths, checks, and current-expression rewrites when those are part of the pencil-and-paper procedure.
- Verify before answering where natural: substitute back, apply an inverse operation, check a magnitude, or emit another compact `CHECK` step before `Z|`.
- Do not require unstated lookups. Any trig value, z/t/chi-square critical value, logarithm, normal CDF value, or other table/calculator value must be supplied in the problem text, avoided by construction, or left symbolic/exact.
- Use hand-friendly operands. The procedure should be the hard part, not digit grinding.
- If the answer space is tiny, make `final_answer` composite enough to grade reliably rather than a coin-flip label.
- Construct data backward from exact answers: use triples, perfect squares, denominators dividing powers of 2 and 5 for `dec()`, dyadic probabilities, divisible coefficients, or exact symbolic forms. `dec(Fraction)` is only valid for terminating decimals; otherwise render a reduced fraction or constrain the inputs.
- Pipe-safety is mandatory: no step field may contain raw ASCII `|`. Use alternatives such as `abs(r)` or `‖u‖`. Keep steps to at most four payload fields after the op-code.

## Coding Style & Naming Conventions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [QuixiAI/QuixiMath](https://github.com/QuixiAI/QuixiMath) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
