---
trigger: always_on
description: This project exists to provide a high-quality PySide6 frontend for ComfyUI with excellent usability, reliability, and maintainability.
---

# AGENTS.md

## Mission Statement

This project exists to provide a high-quality PySide6 frontend for ComfyUI with excellent usability, reliability, and maintainability.
Engineering priority is strict architecture, strong separation of concerns, behavior safety during structural change, and long-term developer velocity.

## Purpose

- This file defines engineering guardrails for this repository.
- This file governs architecture, code quality, typing, testing, observability, and safety.
- Do not use this file for feature specs or product planning.

## Behavior Boundary

- Preserve existing user-facing behavior unless explicitly approved to change.
- Preserve compatibility for persisted files and project data unless explicitly approved to change.
- Treat current behavior and persisted formats as the contract; change internals freely within that boundary.

## Localization Policy

- Route all SugarSubstitute-owned user-facing text, including installer text, through its explicit localization owner. Hard-coded visible copy is not allowed.
- Treat `languages.json` as the sole supported-locale registry. Do not duplicate locale inventories in source, tools, or tests.
- Every release-enabled locale must have complete application and installer coverage. Add or change owned text and all supported translations atomically; add a locale only with complete coverage for every owned surface.
- When one source string is intentionally shared or repeated, update every matching occurrence and catalog entry together. Do not leave inconsistent near-duplicates for later cleanup.
- English is the canonical source and fallback language. Write translations directly and do not use external translation aids.
- Every release-enabled non-English locale in `languages.json` must have a complete `README.<language-id>.md`, and every README language selector must list the full release-enabled set. Treat `README.md` as the authority for README facts, voice, tone, and style. Any change to it must update every localized README in the same change. Keep each version complete and factually aligned, but localize the writing so it creates the same experience for its readers; a mechanical translation is not sufficient.
- Preserve cube-authored and user-authored text exactly. Resolve eligible ComfyUI-owned node text from the connected ComfyUI server and never maintain a parallel local ComfyUI corpus.
- Preserve Unicode and IME input, widget behavior, styling, and layout across locales. A localization change that regresses text entry or presentation behavior is a failure.
- Keep the source-ownership and absolute-coverage gates strict. Fix violations at their owner; do not weaken extraction, parity, completion, or runtime-catalog checks to pass CI.

## Environment and Gate Execution

- All verification commands must run against the repository virtual environment at `.venv`.
- Do not run quality gates with global/system Python.
- If `.venv` is missing or stale, bootstrap with `.\venv.bat` first.
- Run all commands from repo root in PowerShell.

### Required command forms (PowerShell)

- Focused tests: `.\.venv\Scripts\python.exe -m pytest -n auto -q <paths>`
- Full parallel tests: `.\.venv\Scripts\python.exe -m pytest -n auto -q -m "not serial"`
- Full serial tests: `.\.venv\Scripts\python.exe -m tools.ci.run_serial_test_modules --junit-dir=build\test-results\local-serial`
- Lint: `.\.venv\Scripts\ruff.exe check .`
- Format: `.\.venv\Scripts\ruff.exe format .`
- Type check: `.\.venv\Scripts\mypy.exe --strict substitute tests`

## Core Engineering Principles

- Use strict object-oriented design.
- Enforce strong separation of concerns as the primary architecture objective.
- Keep modules cohesive and boundaries explicit.
- Assign one authoritative owner per concern; other components may participate, but must derive their behavior, state, and geometry from that owner rather than re-implementing the concern in parallel.
- Reassess ownership before extending an existing structure; if a change introduces a distinct responsibility, change cadence, or collaboration boundary, split or extract it as part of the change instead of deferring cleanup.
- Prefer clean replacement over compatibility layers in internal code.
- Structural changes must be complete: update callsites, remove dead code, remove temporary bridges.
- Favor DRY when it reduces repeated change risk.
- Avoid abstractions that hide intent.

## Architecture Rules

- Organize code into clear layers with one-way dependencies.
- UI/Presentation layer: widgets, view composition, Qt-specific interaction.
- Application/Orchestration layer: use-case coordination, workflow orchestration, lifecycle control.
- Domain layer: core models, business rules, visibility/activation/policy logic, pure logic where possible.
- Infrastructure/Adapter layer: filesystem, network, subprocess, Comfy integration, plugin/custom-node operations.
- Higher-level layers may depend on lower-level layers; lower-level layers must not depend on higher-level layers.
- Keep Qt types out of domain logic whenever feasible.
- Place code by ownership and dependency direction, not convenience or proximity.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Artificial-Sweetener/SugarSubstitute](https://github.com/Artificial-Sweetener/SugarSubstitute) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
