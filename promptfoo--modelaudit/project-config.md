---
trigger: always_on
description: This is the single source of truth for all AI coding agents (Claude, Gemini, others) working on ModelAudit, a security scanner for AI/ML model files. Follow it exactly and keep instructions concise through progressive disclosure—share only the minimum needed context and iterate.
---

# AGENTS.md — ModelAudit (Canonical Agent Guide)

This is the single source of truth for all AI coding agents (Claude, Gemini, others) working on ModelAudit, a security scanner for AI/ML model files. Follow it exactly and keep instructions concise through progressive disclosure—share only the minimum needed context and iterate.

## Stateless Onboarding

- Agents start with zero context; use this file to bootstrap each session with the essentials: what (stack/project map), why (security-focused scanner), and how (workflow + validation below).
- Prefer pointers over payloads: read the specific docs in `docs/agents/` when needed instead of inlining here.
- Keep instructions universal and minimal; lean on deterministic tools (ruff, mypy, pytest, prettier) rather than embedding style rules.
- When unsure, ask or fetch targeted context instead of expanding instructions.

### Monorepo at a glance

This repo publishes **two PyPI packages with independent versions**:

| PyPI name               | Path                              | Version file                    | CHANGELOG                                     |
| ----------------------- | --------------------------------- | ------------------------------- | --------------------------------------------- |
| `modelaudit`            | `./` (root)                       | `pyproject.toml` + `uv.lock`    | `CHANGELOG.md`                                |
| `modelaudit-picklescan` | `packages/modelaudit-picklescan/` | `pyproject.toml` + `Cargo.toml` | `packages/modelaudit-picklescan/CHANGELOG.md` |

Root `modelaudit` hard-requires `modelaudit-picklescan>=0.1.0,<0.2.0` — when the sibling crosses `0.2.0`, bump the constraint in the same PR or the next `modelaudit` release is uninstallable. Both packages are driven by a single `release-please` workflow (`.github/workflows/release-please.yml`) with components defined in `release-please-config.json` and current versions in `.release-please-manifest.json`. Full publishing details — trusted publishing, manual `workflow_dispatch` recovery (`root_version` / `picklescan_version`), and yank procedure — are in [`docs/agents/release-process.md`](docs/agents/release-process.md). For work inside the picklescan package, start from [`packages/modelaudit-picklescan/AGENTS.md`](packages/modelaudit-picklescan/AGENTS.md).

## Mission & Principles

- **Security first:** Never weaken detections or bypass safeguards.
- **Match the codebase:** Follow existing patterns, architecture, and naming; never add dependencies without approval.
- **Progressive disclosure:** Be concise, reveal details as needed, and prefer short, scoped messages.
- **Iterative refinement:** Share a plan for non-trivial work, execute incrementally, and verify after each change.
- **Ask when unclear:** Confirm scope before risky or ambiguous actions.
- **Proactive completion:** Provide tests and follow-up steps without waiting to be asked.

## Quick Start Commands

```bash
# Setup
uv sync --extra all-ci

# Pre-commit workflow (MUST run before every commit)
uv run ruff format modelaudit/ packages/modelaudit-picklescan/src packages/modelaudit-picklescan/tests tests/
uv run ruff check --fix modelaudit/ packages/modelaudit-picklescan/src packages/modelaudit-picklescan/tests tests/
uv run mypy modelaudit/ packages/modelaudit-picklescan/src packages/modelaudit-picklescan/tests tests/
PROMPTFOO_DISABLE_TELEMETRY=1 uv run pytest -n auto -m "not slow and not integration" --maxfail=1
```

## Standard Workflow

1. **Understand:** Read nearby code, tests, and docs (`docs/agents/*.md`) before editing.
2. **Plan:** For anything non-trivial, present a short multi-step plan; refine iteratively.
3. **Implement:** Preserve security focus, follow `BaseScanner` patterns (see `docs/agents/architecture.md`), handle missing deps gracefully, and update `SCANNER_REGISTRY` when adding scanners.
4. **Verify:** Run the validation commands above. Format/linters must be clean. Use targeted `pytest` when appropriate, and type-check modified tests as part of the normal `mypy modelaudit/ packages/modelaudit-picklescan/src packages/modelaudit-picklescan/tests tests/` pass.
5. **Report:** Summarize changes with file references and note residual risks or follow-ups.

## Branch & Git Hygiene

**NEVER commit or push directly to `main`.** All changes must go through pull requests.

```bash
# Start clean
git fetch origin main
git checkout main
git merge --no-edit origin/main

# Work on a branch (REQUIRED - never commit to main)
git checkout -b feat/your-feature-name  # or fix/, chore/, test/

# Commit (conventional)
git commit -m "feat: add scanner for XYZ format

Description here."

# PR (after validation) - ALL changes go through PRs
git push -u origin feat/your-feature-name
gh pr create --title "feat: descriptive title" --body "Brief description"
```

- Use non-interactive flags (`--no-edit`, `-m`). One command per invocation; avoid long `&&` chains.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [promptfoo/modelaudit](https://github.com/promptfoo/modelaudit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
