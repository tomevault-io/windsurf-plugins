---
trigger: always_on
description: Guidance for AI agents and human contributors working on this repository.
---

# AGENTS – Mindstone Rebel

Guidance for AI agents and human contributors working on this repository.

> **Public-mirror note:** This is the public-mirror version of `AGENTS.md`. It is maintained separately from Mindstone's internal development copy so internal workflow details do not ship in the OSS repo. References to `docs/plans/`, `coding-agent-instructions/`, or `private/` may point to internal-only material; treat them as context, not required public dependencies.


## Project Overview

- **App**: User-friendly agentic-AI Electron app powered by **Rebel Core**, Rebel's native in-process agent runtime (direct provider APIs + MCP, no agent subprocess for the main turn). See [REBEL_CORE](docs/project/REBEL_CORE.md) and [ARCHITECTURE_AGENT_TURN_EXECUTION](docs/project/ARCHITECTURE_AGENT_TURN_EXECUTION.md) for the execution model.
- **Target audience**: **Non-technical knowledge workers**: executives, product managers, sales and marketing teams, customer-success teams, researchers, and other professionals. Product decisions should be evaluated through that lens. Users think in terms of meeting prep, email triage, research synthesis, and document drafting — not implementation details.
- **Stack**: Electron + React + TypeScript + Vite (via `electron-vite`).
- **Platforms**: Cross-platform desktop (macOS, Windows, Linux), with shared cloud and mobile surfaces. Keep code and user-facing copy platform-agnostic. See [WINDOWS_SUPPORT](docs/project/WINDOWS_SUPPORT.md) for path, process, and packaging gotchas.


## Brand Voice & Product Philosophy

See [BRAND_VOICE](docs/project/BRAND_VOICE.md) for the full voice guide.

Quick summary: Rebel is dry, witty, and self-aware — a capable colleague who happens to be amusing. Bias toward clear over clever, calm over exciting, and useful over impressive. Use that voice when touching UI copy, errors, onboarding, notifications, changelogs, and other user-facing surfaces.


## How to Work

### Default operating loop

1. **Understand the task**: read the relevant README/project docs, nearby `AGENTS.md`, and code before editing.
2. **Plan the smallest useful change**: identify files, validation commands, risks, and rollback path.
3. **Implement surgically**: match existing patterns, preserve contracts, and avoid broad rewrites unless the task explicitly calls for one.
4. **Verify**: run the fastest relevant checks during iteration and the project validators before handing off.
5. **Report clearly**: summarize what changed, what passed, and any remaining blockers.

### AI-agent workflows

For non-trivial work, use a structured workflow even if your local tooling names it differently:

- **Feature/refactor work**: create a staged implementation plan, review boundary impacts, implement one coherent stage at a time, and get a second-model or peer review when available. Internal workflow docs may call this `CHIEF_ENGINEER`.
- **Bug fixes**: reproduce or localize the failure, compare plausible root causes, make a minimal fix, add or update regression coverage, and record the learning. Internal workflow docs may call this `CHIEF_BUGFIXER`.
- **Renames**: enumerate all variants and surfaces before editing. Concept renames are broader than file renames; include UI copy, persisted data, prompts, tests, and docs.

Trivial changes (single-file copy edits, small config tweaks, obvious documentation fixes) can be handled directly.


## Coding Principles Digest

### Mandatory stop-and-check gates

When one of these applies, pause and read the linked project guidance before editing:

- **UI work**: use shared UI components from `src/renderer/components/ui`, follow [UI_OVERVIEW](docs/project/UI_OVERVIEW.md), [UI_CSS_ARCHITECTURE](docs/project/UI_CSS_ARCHITECTURE.md), and [BRAND_VOICE](docs/project/BRAND_VOICE.md). Test light and dark themes when changing visuals.
- **E2E test failures**: read [E2E_TEST_FIXING_GUIDELINES](docs/project/E2E_TEST_FIXING_GUIDELINES.md) and [WHY_E2E_TESTS_ARE_HARD_TO_FIX](docs/project/WHY_E2E_TESTS_ARE_HARD_TO_FIX.md). Do not weaken coverage to make a failure disappear.
- **Version bumps**: follow [CHANGELOG_UPDATE_PROCESS](docs/project/CHANGELOG_UPDATE_PROCESS.md) and update the required changelogs before or alongside the bump.
- **LLM prompt changes**: update eval fixtures and run the relevant eval suite. See [WRITING_EVALS](docs/project/WRITING_EVALS.md).
- **New eval harnesses**: read [WRITING_EVALS](docs/project/WRITING_EVALS.md) in full before designing the harness.
- **Model/provider/auth/routing work**: start with [MODEL_AND_PROVIDER_OVERVIEW](docs/project/MODEL_AND_PROVIDER_OVERVIEW.md).
- **IPC changes**: every new handler needs a Zod contract in `src/shared/ipc/contracts.ts`. See [ARCHITECTURE_IPC](docs/project/ARCHITECTURE_IPC.md).
- **Cross-process or connector boundaries**: check [BOUNDARY_REGISTRY](docs/project/BOUNDARY_REGISTRY.md) and run the boundary-hint tooling if contracts, MCP connectors, IPC schemas, or shared boundary interfaces change.
- **Destructive git/file operations**: never discard, reset, or delete work you did not create. Treat untracked files as user-owned.
- **Secrets**: never commit keys, tokens, DSNs, customer data, or private URLs. See [SECRET_SCANNING](docs/project/SECRET_SCANNING.md).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mindstone/rebel-app](https://github.com/mindstone/rebel-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
