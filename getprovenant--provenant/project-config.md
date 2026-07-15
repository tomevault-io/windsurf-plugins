---
trigger: always_on
description: This file is for evergreen repo-specific guardrails and recurring agent gotchas. It intentionally duplicates a small set of high-value facts from the canonical docs so those facts stay in agent context even when an agent does not read further. Keep long procedures and fast-changing workflow detail in the canonical docs instead of fully restating them here.
---

# Agent Guidelines for Provenant

This file is for evergreen repo-specific guardrails and recurring agent gotchas. It intentionally duplicates a small set of high-value facts from the canonical docs so those facts stay in agent context even when an agent does not read further. Keep long procedures and fast-changing workflow detail in the canonical docs instead of fully restating them here.

## Start Here

- [`README.md`](README.md) for the project overview, user-facing setup, and CLI entry points.
- [`CONTRIBUTING.md`](CONTRIBUTING.md) for the main contributor workflow, local setup, hooks, commit/PR conventions, DCO, and license-header policy.
- [`docs/DOCUMENTATION_INDEX.md`](docs/DOCUMENTATION_INDEX.md) when you need to find which document owns a topic.

Important baseline facts to remember on every run:

- `README.md` is user-facing; `CONTRIBUTING.md` is the main contributor workflow document.
- Contributor setup is dual-stack: Rust toolchain plus Node.js `>=24` with `npm`.
- The usual local bootstrap is `npm run setup`.
- `npm run hooks:run` runs the full pre-commit hook suite on all files.
- `npm run check:docs` is the default validation entry point for documentation-only changes.

Before making non-trivial changes, read the document that owns the surface you are touching:

- [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) for overall system design, parser/assembly boundaries, and scanner-owned exceptions.
- [`docs/LICENSE_DETECTION_ARCHITECTURE.md`](docs/LICENSE_DETECTION_ARCHITECTURE.md) for license-index, dataset, cache, and detection-pipeline details.
- [`docs/HOW_TO_ADD_A_PARSER.md`](docs/HOW_TO_ADD_A_PARSER.md) for parser workflow, registration, datasource wiring, and assembly/file-reference integration.
- [`docs/TESTING_STRATEGY.md`](docs/TESTING_STRATEGY.md) for test layers and validation expectations.
- [`xtask/README.md`](xtask/README.md) for maintainer workflows such as compare runs, golden maintenance, generated artifacts, and benchmark helpers.

## Project Context

Provenant's primary goal is to produce the best practical scan result for users: accurate, explainable, bounded, and maintainable. ScanCode compatibility is an important compatibility lane and regression signal, not the end goal. Preserve parity where users depend on it, but do not treat ScanCode output or implementation choices as automatically correct when Provenant can provide a better-supported result.

Routine scans use the embedded license index. The `reference/scancode-toolkit/` submodule is mainly needed for parity research, embedded-license-data maintenance, and maintainer workflows that depend on upstream material.

Use the Python ScanCode codebase as a behavioral reference, not an implementation template or an unquestioned source of truth. When comparing with ScanCode code or scan output, first identify the user-facing contract it demonstrates, then ask whether Provenant can solve the case more accurately, clearly, or safely before adopting the same behavior.

When an upstream test fixture is needed for Provenant tests, copy it into Provenant-owned `testdata/` and reference that local copy. Do **not** make tests or golden fixtures depend directly on paths under `reference/scancode-toolkit/`. Prefer synthetic or truncated fixtures over wholesale copies. When you do add a **substantial, wholesale** copy of an identifiable third-party file or package — especially under a copyleft or notice-required license — record its source and license in a short co-located note (a `README`/`SOURCE` in the containing directory), not in a central index. See [`testdata/PROVENANCE.md`](testdata/PROVENANCE.md) for the overall provenance policy.

## Run-Every-Time Guardrails

- Keep parsing static and bounded. Do not execute package-manager code, project code, or shell commands to recover metadata.
- Preserve behavior and parity where users depend on it. If Provenant intentionally diverges, preserve and test the explicit Provenant contract and any documented compatibility lane.
- Treat ScanCode deltas as evidence to investigate, not automatic Provenant bugs. When a difference improves correctness, precision, safety, explainability, or boundedness, keep the Provenant behavior and document or test the contract as appropriate.
- Keep package extraction separate from broader detection work. Parsers may normalize trustworthy declared package-license metadata, but file-content license and copyright detection belong to the detection pipeline.
- Parsers are file-local extractors. Cross-file ownership, topology-aware workspace handling, and file-reference resolution belong in assembly unless an existing documented scanner-owned exception says otherwise.
- Prefer honest unknowns over guessed compatibility defaults. If a datasource does not prove dependency intent such as `is_runtime`, `is_optional`, `is_direct`, or `is_pinned`, leave it unset.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [getprovenant/provenant](https://github.com/getprovenant/provenant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
