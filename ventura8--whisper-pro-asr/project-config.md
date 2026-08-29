---
trigger: always_on
description: - **Mandatory Pre-Task Agent Review**: Before starting any implementation, read `.agent/instructions.md`, `.agent/skills/SKILLS_CATALOG.md`, and every directly relevant skill/workflow file. Do not begin coding until this review is complete.
---

# AI Instructions

## Global Rules

- **Mandatory Pre-Task Agent Review**: Before starting any implementation, read `.agent/instructions.md`, `.agent/skills/SKILLS_CATALOG.md`, and every directly relevant skill/workflow file. Do not begin coding until this review is complete.
- **Mandatory Markdown Update (Every Task, No Exceptions)**: Every time you work on this project, you MUST update all relevant markdown files in the same task. Never ship code-only changes. This applies to every kind of change, including CI/CD, Docker, and other infrastructure/tooling work, not just application code. Before closing work, sync every impacted file among `README.md`, `docs/*.md`, and `.agent/*.md` so they describe the current implementation. A task is incomplete until relevant docs are updated.
- **Concurrency Correctness Priority #1**: For scheduler/resource/lifecycle work, deadlock/livelock prevention and bounded progress are mandatory and take precedence over throughput optimizations.
- **Preserve Helpful Comments**: Never delete comments that explain logic, especially around synchronization, error handling, or non-obvious code paths. Code clarity and maintainability take precedence over aggressive line count reduction.
- **Strict File Size Limit**: Any `.py` file MUST NOT exceed **500 lines**.
- **Logging Only**: Use `logger` (logging module) for all output. No `print()` calls allowed.
- **Hardware Agnostic**: Ensure code works across NPU, GPU (Intel/NVIDIA), and CPU. Use `modules.core.config` for device selection.
- **Agent Asset Maintenance (Mandatory)**: Whenever code, architecture, CI flow, testing strategy, release process, or operational behavior changes, update all affected agent assets in `.agent/` during the same task. This includes `instructions.md`, relevant files in `.agent/skills/`, and `.agent/workflows/` so agent guidance stays accurate.
- **Frontend Gate Maintenance (Mandatory)**: Whenever dashboard HTML/JS/CSS changes, run and keep aligned the frontend quality gates (`npm run quality:frontend`) and update related documentation/skills if commands or thresholds change.
- **Frontend Security Gate (Mandatory)**: CI and local build parity scripts must fail on any npm audit vulnerability using `npm audit --audit-level=low` after `npm ci`.
- **Build Script Bootstrap (Mandatory)**: `scripts/ci/build-and-test.sh` and `scripts/ci/build-and-test.ps1` are allowed to bootstrap missing `npm`/`docker` dependencies on Linux via `apt-get`, but must fail clearly when automatic installation is unavailable.
- **Playwright + MCP Tooling (Mandatory)**: For frontend and dashboard verification, use Playwright CLI commands (`npx playwright ...` or npm scripts wrapping Playwright CLI) and MCP browser tooling for DOM/state inspection and deterministic troubleshooting. Do not rely on ad-hoc browser/manual-only verification paths when these automated tools are available.
- **Docker-Only Quality Execution**: All linting, formatting, schema validation, type checking, dependency auditing, security scanning, and testing MUST be executed exclusively within the Docker test image pipeline (`Dockerfile.test` target `test`). Host-side executions of these quality gates are forbidden; the local/CI wrappers (`build-and-test.sh` and `build-and-test.ps1`) must only build and run the Docker test image and consume its results.
- **PR Review-Thread Closure (Mandatory)**: After fixing (or determining no change is needed for) a PR review comment/thread, reply to that specific thread on GitHub explaining the outcome (fixed / already resolved / not applicable / deferred, with a brief reason), then mark it resolved. Do this for every thread addressed in the pass, not just a single summary comment on the PR. Only reply/resolve once the underlying fix is actually pushed and visible in the PR diff — do not resolve a thread against a local, unpushed commit.

## Task & Status Display Priority (Mandatory)

Correct task status display on the dashboard is a critical user-facing contract.
Any change to scheduling, preemption, or monitoring MUST preserve correctness and requires synchronized agent asset and test updates.

### Mandatory Checklist

- **Status Enum Stability**: The 7 task statuses (`initializing`, `queued`, `active`, `post-processing`, `completed`, `failed`, `unknown`) are immutable. Do not add, remove, or rename without corresponding updates to `.agent/skills/monitoring/task_status_display_specification_skill.md`, frontend tests, and backend telemetry tests. Note: `unknown` is an **internal-only fallback** status; it must be normalized to a concrete canonical label (e.g., `initializing`) before any API payload serialization or dashboard rendering — it must never appear as a visible display value.
- **Stage Transitions**: When adding new pipeline stages or modifying preemption behavior, update `.agent/skills/monitoring/task_status_display_specification_skill.md` with the new stage semantics. Verify dashboard rendering reflects the change and tests validate the transition.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ventura8/Whisper-Pro-ASR](https://github.com/ventura8/Whisper-Pro-ASR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
