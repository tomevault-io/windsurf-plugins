---
trigger: always_on
description: This file is a pointer board for agents working in this repository. Keep it
---

# Agent Guidelines for DART

This file is a pointer board for agents working in this repository. Keep it
concise and expand other documents instead.

## Project Overview

**WHAT**: C++23 physics engine for robotics simulation with Python bindings (dartpy)
**WHY**: Research-grade dynamics for robotics, animation, and machine learning
**HOW**: Build/test with `pixi run` tasks; format with `pixi run lint` before commits

## Quick Commands

```bash
pixi run test-all       # Lint + build + all tests (fails fast)
pixi run lint           # Format code/docs (auto-fixes)
pixi run build          # Build C++ and Python
pixi run test-unit      # Unit tests only (faster)
pixi run test-py        # Python tests only
pixi run -e cuda test-all # CUDA full validation on Linux CUDA hosts
pixi run check-lint     # Check formatting without fixing
pixi run ai-doctor      # Read-only agent/setup diagnosis
pixi run check-ai-infra # AI discovery, drift, hook, and scenario checks
```

Success: "All tests passed!"

If this fails, see `docs/onboarding/ci-cd.md` for troubleshooting.

## Context Loading (IMPORTANT)

**Agents MUST load relevant docs before starting work.** Start every session by
reading `docs/ai/principles.md`, then load the task-specific docs below. Tools
that support `@file` references may use them, but the paths below are the
portable source of truth.

| Task Type                                | Load These Files                                                                                                                                                                              |
| ---------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Any task**                             | This file (auto-loaded), `docs/ai/principles.md`                                                                                                                                              |
| Project direction                        | `docs/ai/north-star.md`                                                                                                                                                                       |
| Building                                 | `docs/onboarding/building.md`                                                                                                                                                                 |
| Testing                                  | `docs/onboarding/testing.md`                                                                                                                                                                  |
| Model / simulation / visual verification | `docs/onboarding/agent-sim-verification.md`, `docs/ai/verification.md`; use `/dart-verify-sim` or `$dart-verify-sim`                                                                          |
| Contributing                             | `docs/onboarding/contributing.md`, `CONTRIBUTING.md`                                                                                                                                          |
| Code style                               | `docs/onboarding/code-style.md`                                                                                                                                                               |
| Docs structure / placement               | `docs/README.md`, `docs/information-architecture.md`, `docs/AGENTS.md`                                                                                                                        |
| Architecture                             | `docs/onboarding/architecture.md`, `docs/onboarding/README.md`                                                                                                                                |
| DART 7 architecture vision               | `docs/readthedocs/architecture.md` (multi-physics/solver/backend one-paper), `docs/design/simulation_solver_architecture.md`                                                                  |
| Architecture hardening / work packets    | `docs/design/dart7_architecture_assessment.md` (verified findings + standing rule), `docs/plans/solver-family-intake.md`, `docs/ai/orchestration.md`                                          |
| Public API work                          | `docs/onboarding/api-boundaries.md`                                                                                                                                                           |
| Theory/research foundations              | `docs/background/README.md`                                                                                                                                                                   |
| CI/CD issues                             | `docs/onboarding/ci-cd.md`                                                                                                                                                                    |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dartsim/dart](https://github.com/dartsim/dart) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
