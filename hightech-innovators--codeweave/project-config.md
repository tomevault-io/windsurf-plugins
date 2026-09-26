---
trigger: always_on
description: CodeWeave is a GitHub Actions automation system that orchestrates iterative code improvements using the GitHub Copilot CLI. It clones an external repository and runs Copilot across a documentation pipeline — book generation, ADR generation, performance measurement harness design, integration test code generation — then a measurement-and-optimization pipeline: source build (5), baseline execution (6), autonomous optimization cycles with an A/B verdict per change (7), and an aggregate report (8). 
---

# Copilot Instructions for CodeWeave

## Project Overview

CodeWeave is a GitHub Actions automation system that orchestrates iterative code improvements using the GitHub Copilot CLI. It clones an external repository and runs Copilot across a documentation pipeline — book generation, ADR generation, performance measurement harness design, integration test code generation — then a measurement-and-optimization pipeline: source build (5), baseline execution (6), autonomous optimization cycles with an A/B verdict per change (7), and an aggregate report (8). Each generative phase stops early when its completion artifact is produced.

The flow runs as **GHA workflows**: `codeweave.yml` for Phases 1–6; `phase-7-optimize.yml` for the auto-chaining Phase 7 cycles; `phase-8-report.yml` for Phase 8.

**Key Components:**
- **Workflow:** `.github/workflows/codeweave.yml` — orchestrator for Phases 1–6; it calls per-phase reusable workflows (`.github/workflows/phase-*.yml`) and shares bootstrap via the `.github/actions/codeweave-setup` composite action
- **Optimization workflow:** `.github/workflows/phase-7-optimize.yml` — One optimization per run, auto-chaining (Phase 7); `.github/workflows/phase-8-report.yml` — aggregate report (Phase 8)
- **Configuration:** `.github/codeweave.config` — Environment variables for external repo, branch, iterations, model schedules, and git identity
- **Phase 1 Work Definition:** `work/1-generate-book.md` / `work/1-validate-book.md` — Book generation and validation prompts
- **Phase 2 Work Definition:** `work/2-generate-adrs.md` / `work/2-validate-adrs.md` — ADR generation and validation prompts
- **Phase 3 Work Definition:** `work/3-generate-harness.md` / `work/3-validate-harness.md` — Performance measurement harness design and validation prompts
- **Phase 4 Work Definition:** `work/4-generate-tests.md` / `work/4-validate-tests.md` — Integration test code generation and validation prompts
- **Phase 6 Work Definition:** `work/6-repair-tests.md` — repair agent prompt (fix runtime errors in `tests/` and `_tools/`). Baseline collection itself is deterministic pipeline logic (`run.sh` × N → `ab_compare.py --mode baseline`), not a Copilot prompt
- **Constraints:** `constraints/project.md` — Repository-specific constraints and requirements (customize for your fork)
- **Perf Constraints:** `constraints/harness.md` — *(optional)* Target execution constraints for Phase 3 (hardware, scope, time budgets, isolation)
- **Perf Context:** `constraints/harness-context.md` — *(optional)* Domain context for Phase 3 scenario and observability design (also read by Phase 4 to extract the scenario prompt set)
- **Proof Directory:** `proof/` — Auto-generated output artifacts from each Copilot run

## Architecture

Phases 1–4 follow a **generate+validate loop pattern**; Phases 5–8 (the measurement-and-optimization extension) are documented after Finalization below:

1. **Initialization**: Clones external repository into `src` on the specified branch, then creates and checks out the work branch (excluded from git tracking)

2. **Phase 1 — Book Generation** (up to `PHASE1_MAX_ITERATIONS`):
   - Each iteration is a generate pass followed by a validate pass
   - Generate: invokes Copilot with a prompt to work on `work/1-generate-book.md` / `constraints/project.md` and improve `./src`
   - Validate: invokes Copilot with a prompt to validate per `work/1-validate-book.md` — the validator is the **only** agent allowed to write `book/manuscript-complete.md`
   - Allowed tools: the full toolset except `shell(git:*)` (`--allow-all-tools --deny-tool='shell(git:*)'`) — the pipeline commits after each pass, so the agent is never granted git
   - Captures output to `proof/1-book-generation-N.md` / `proof/1-book-generation-session-N.md` (generate) and `proof/1-book-validation-N.md` / `proof/1-book-validation-session-N.md` (validate)
   - Commits changes to the outer repo after each pass
   - Generator writes AI working-state files to `agent-state/` in the target repo (created on first run)
   - After Phase 1, builds `book.pdf` (Pandoc → Typst) and generates `book/BOOK-INDEX.md`
   - **Early Exit**: Stops if `book/manuscript-complete.md` is present after a validate pass

3. **Phase 2 — ADR Generation** (up to `PHASE2_MAX_ITERATIONS`, only if Phase 1 produced a PDF):
   - Each iteration is a generate pass followed by a validate pass
   - Generate: invokes Copilot with a prompt to work on `work/2-generate-adrs.md` / `constraints/project.md`, using `./book` as reference
   - Validate: invokes Copilot with a prompt to validate per `work/2-validate-adrs.md` — the validator is the **only** agent allowed to write `src/adrs-complete.md`
   - After each iteration, commits and pushes ADR changes directly to the target repo's work branch

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HighTech-Innovators/CodeWeave](https://github.com/HighTech-Innovators/CodeWeave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
