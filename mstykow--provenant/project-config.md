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

Provenant is an independent Rust implementation for ScanCode-aligned workflows. The goal is strong compatibility users can trust, while using Rust to improve correctness, safety, performance, and maintainability.

Routine scans use the embedded license index. The `reference/scancode-toolkit/` submodule is mainly needed for parity research, embedded-license-data maintenance, and maintainer workflows that depend on upstream material.

Use the Python ScanCode codebase as a behavioral specification, not as an implementation template. Preserve behavior and edge cases, but write the Rust implementation idiomatically.

When an upstream test fixture is needed for Provenant tests, copy it into Provenant-owned `testdata/` and reference that local copy. Do **not** make tests or golden fixtures depend directly on paths under `reference/scancode-toolkit/`.

## Run-Every-Time Guardrails

- Keep parsing static and bounded. Do not execute package-manager code, project code, or shell commands to recover metadata.
- Preserve behavior and parity where users depend on it. If Provenant intentionally diverges, preserve and test the explicit Provenant contract and any documented compatibility lane.
- Keep package extraction separate from broader detection work. Parsers may normalize trustworthy declared package-license metadata, but file-content license and copyright detection belong to the detection pipeline.
- Parsers are file-local extractors. Cross-file ownership, topology-aware workspace handling, and file-reference resolution belong in assembly unless an existing documented scanner-owned exception says otherwise.
- Prefer honest unknowns over guessed compatibility defaults. If a datasource does not prove dependency intent such as `is_runtime`, `is_optional`, `is_direct`, or `is_pinned`, leave it unset.
- Use `cargo add`, `cargo remove`, and targeted `cargo update` instead of editing Rust dependencies by hand. Do not add dependencies lightly, and check maintenance health before introducing a new one.
- Treat contributor tooling as dual-stack: Rust plus Node/npm-managed hooks and doc tooling. See [`CONTRIBUTING.md`](CONTRIBUTING.md) and [`package.json`](package.json) for the current bootstrap and helper commands.

## Frequent Agent Gotchas

- Scan commands need at least one explicit output flag such as `--json-pp -` or `--json out.json`.
- Detections are opt-in. Flags such as `--license`, `--package`, `--copyright`, `--info`, `--email`, and `--url` change what the scan actually collects.
- Default file-level copyright output is more source-faithful than historic ScanCode rendering; use `--compat-mode scancode` when a parity-sensitive workflow needs the ScanCode-style rendered value.
- `--package-only` is not a synonym for `--package`; it is a narrower fast path with different output semantics.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mstykow/provenant](https://github.com/mstykow/provenant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
