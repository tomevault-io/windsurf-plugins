---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

This is a provider-agnostic 13-phase (0–12) development pipeline for Codex and
Claude Code. It transforms a task description into reviewed, optionally
committed code. The single engine is `run-pipeline.sh`.

## The One Engine

`run-pipeline.sh` is the single real executable. It runs each phase as a fresh
`codex exec --ephemeral` or `claude -p --no-session-persistence` subprocess,
persists the final report, validates it, applies the gate, and optionally
commits after Phase 12.

- **`run-pipeline.sh`** — the engine. Run it directly: `bash run-pipeline.sh [options] "task"`.
- **`.claude/commands/auto-pipeline.md`** — a thin Claude `/auto-pipeline` wrapper that
  runs the engine with `PIPELINE_NONINTERACTIVE=1` and interprets its exit code.
- Codex runs the engine directly. The files under `.codex/agents/` are optional
  interactive helpers and are not used by the engine.

## Commands

```bash
# Run the pipeline (standalone)
bash run-pipeline.sh "add a GET /api/version endpoint"
bash run-pipeline.sh --provider=codex "add a GET /api/version endpoint"
bash run-pipeline.sh --provider=claude "add a GET /api/version endpoint"
bash run-pipeline.sh --profile=paranoid --mode=dev "handle payments"

# Demo starter project (demo/starter-project/)
npm install && npm test
```

Flags the engine actually parses: `--provider=auto|claude|codex`,
`--profile=yolo|fast|standard|paranoid`, `--mode=auto|dev`,
`--skip-arm` (skip Phase 1), `--skip-ar` (skip Phase 3), `--skip-pmatch` (skip Phase 5),
`--model-strong=`, `--model-fast=`, `--max-budget-usd=` (per-phase cap), `--max-run-budget-usd=`
(whole-run cap), `--no-commit`, `--allow-dirty`, `--allow-untested-commit`,
`--resume=RUN_ID`, `--policy-rollout=legacy|shadow|enforced`,
`--retention-days=`, `--retention-max-runs=`, `--help`, plus `--push` (publish the committed
run branch to the remote) and `--pr` (`--push` plus PR guidance). Resume requires the original
task and an exact engine/config/Git/evidence match. Still not implemented:
`--template`, `--batch-qa`, `--fix`, and a `--yolo` shorthand.

## Architecture

### The 13-Phase Pipeline

```
Phase 0:  Pre-Check          (HARD) → Find existing code/libraries before building
Phase 1:  Requirements       (SOFT) → Extract testable success criteria
Phase 2:  Design             (SOFT, STRONG model) → Architecture decisions with citations
Phase 3:  Adversarial Review (HARD, STRONG model) → 3 critic angles stress-test the design
Phase 4:  Planning           (SOFT) → Exact BEFORE/AFTER code for every change
Phase 5:  Drift Detection    (SOFT) → Verify the plan covers the design
Phase 6:  Build              (HARD) → Execute the plan; halt if blocked
Phase 7:  Denoise            (NONE) → Strip debug artifacts / dead code
Phase 8:  Quality Fit        (NONE) → Types, lint, conventions
Phase 9:  Quality Behavior   (SOFT) → Gates on the REAL captured test exit code (un-fakeable)
Phase 10: Quality Docs       (NONE) → Swagger/JSDoc coverage
Phase 11: Security           (HARD) → Non-waivable deterministic scanners, then OWASP review
Phase 12: Commit Code-Review (HARD, STRONG model) → Review the real git diff, then commit on APPROVE
```

### Gate System

- **HARD gates** (0, 3, 6, 11, 12): must pass or the pipeline halts for a human (exit 3 when headless).
- **SOFT gates** (1, 2, 4, 5, 9): warn and proceed in `mixed`/`soft` mode; pause in `hard` (paranoid) mode.
- **NONE gates** (7, 8, 10): always proceed; issues are auto-fixed in place.

Phase 9's gate is driven by the **real exit code** of the project's test command, which the
orchestrator (not a model) runs and captures — the one signal a phase cannot fake.

### Model Routing (Balanced)

Model tier and reasoning effort are routed independently.

- Codex: strong `gpt-5.6-sol`, balanced `gpt-5.6-terra`; Security and final
  review use Sol/xhigh.
- Claude: strong `claude-opus-4-8`, balanced `claude-sonnet-5`; Design,
  Adversarial, and final review use Opus/high.
- Neither provider uses `max` by default.
- Routing policy `1.0` records every selection before invocation. It uses only
  explicit task risk/ambiguity evidence: `fast` promotes high-risk Build and
  Security; `standard` also promotes high-risk Requirements and Planning plus
  ambiguous Requirements/Planning; `paranoid` promotes Requirements, Planning,
  Drift Detection, Build, and Security. `yolo` remains fixed except for
  non-skippable high-risk Security.
- Phases 7, 8, and 10 run deterministic checks first. A clean result records
  `SKIP_MODEL`; findings or unavailable checks permit one balanced-lane
  remediation call followed by a deterministic post-check.

### Context: per-phase tool scoping

Claude production subprocesses require `--bare`, load only the built-in tools
their phase needs, use an empty settings-source set plus `--strict-mcp-config`,
and disable CLAUDE.md/auto-memory/background features. Codex production
subprocesses require `--ignore-user-config`, suppress project-document loading,
reject a repository `.codex/config.toml`, disable supported
plugin/memory/subagent features, and enforce read-only/workspace-write

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheAstrelo/Claude-Pipeline](https://github.com/TheAstrelo/Claude-Pipeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
