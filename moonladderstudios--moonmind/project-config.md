---
trigger: always_on
description: Read relevant documents in the following order before implementing tasks:
---

# Agent Instructions

## Read Documentation

Read relevant documents in the following order before implementing tasks:

1. **Constitution:** `.specify/memory/constitution.md` for non-negotiable principles and constraints
2. **Standards:** Code style and guidance in `README.md`
3. **Spec:** `specs/<feature-id>/spec.md`, then `plan.md`, then `tasks.md`
4. **Docs:** `docs/*.md` as needed for system architecture (see **Documentation: canonical vs feature artifacts** below).
   - Start here for Agent Skills: `docs/Tasks/AgentSkillSystem.md`
   - For Executable Tools: `docs/Tasks/SkillAndPlanContracts.md`
   - For Runtime boundaries: `docs/Temporal/ManagedAndExternalAgentExecutionModel.md`
5. **Migration / implementation backlog (when relevant):** MoonSpec artifacts under `specs/<feature>/` and local-only handoffs (for example under `artifacts/`), not disposable paths under `docs/`.

## Agent Skill System Terminology
- Executable `tool.type = "skill"` contracts are **not** the same thing as agent instruction bundles (skill sets) under `.agents/skills`.
- For agent instruction bundles and snapshot logic, the canonical design is in `docs/Tasks/AgentSkillSystem.md`.
- For executable tool contracts, the canonical design is in `docs/Tasks/SkillAndPlanContracts.md`.

## When Modifying the Agent Skill System
When writing code that interacts with skills:
- Read `docs/Tasks/AgentSkillSystem.md`.
- Keep `.agents/skills` as the canonical active path.
- Keep `.agents/skills/local` as a local-only overlay.
- Do not mutate checked-in skill folders in place.
- Keep large skill content out of workflow history (use refs).
- Add workflow/activity or adapter-boundary tests.

## Documentation: canonical vs feature artifacts

- **Canonical docs** (`docs/`): describe **declarative desired state** — architecture, contracts, operator-visible behavior, target semantics. Avoid making phased migration or implementation checklists the main story in these files.
- **Migration, rollout, and MoonSpec execution notes** belong under **`specs/<feature>/`** (and similar feature-local artifacts) or in **local-only / gitignored paths** (e.g. `artifacts/` for tool handoffs), not as the primary framing of canonical docs.
- Align with **Constitution principle XII** in `.specify/memory/constitution.md`.

## Spec Numbering

When creating a new spec folder/feature ID:

- ✅ **DO** scan `specs/` and find the highest numeric prefix across all directories matching `<number>-<name>`.
- ✅ **DO** assign the next number globally (`max + 1`), regardless of short-name/topic.
- ✅ **DO** keep branch/feature/spec numbering aligned to that global next number.
- ❌ **DON'T** reset numbering to `001` for a new short-name if higher numbered specs already exist.

## Testing Instructions

### Test Taxonomy

MoonMind uses a four-tier test model that separates hermetic CI from credentialed provider checks:

| Tier | Marker(s) | Required on PR? | Runner |
|------|-----------|-----------------|--------|
| **Unit** | `asyncio` (as needed) | Yes | `./tools/test_unit.sh` |
| **Hermetic Integration CI** | `integration` + `integration_ci` | Yes | `./tools/test_integration.sh` |
| **Provider Verification** | `provider_verification` + `jules` + `requires_credentials` | No (manual/nightly) | `./tools/test_jules_provider.sh` |
| **Local-only Integration** | `integration` without `integration_ci` | No | local dev only |

- **Hermetic Integration Tests** — compose-backed, local-dependencies-only, no external credentials required.
  These are marked with `@pytest.mark.integration_ci` and are run by the required CI pipeline.
  Use `./tools/test_integration.sh` (Bash) or `tools/test-integration.ps1` (PowerShell) to run them locally.

  The required integration_ci suite focuses on the highest-risk seams:
  - **Artifacts**: create/upload/list, auth/preview, lifecycle cleanup, authorization boundaries
  - **Worker topology**: activity family routing, task queue assignment, sandbox execution
  - **Live logs**: SSE publisher/subscriber, performance at volume, managed runtime streaming
  - **Compose foundation**: service topology, namespace bootstrapping, visibility schema rehearsal
  - **Startup seeding**: profiles, managed secrets, task templates

- **Provider Verification Tests** — real third-party provider checks using real credentials.
  These are **not** required for merge and are excluded from the required CI pipeline.
  They are marked with `@pytest.mark.provider_verification` (and often `@pytest.mark.jules` / `@pytest.mark.requires_credentials`).
  Use `./tools/test_jules_provider.sh` (Bash) or `tools/test-provider.ps1` (PowerShell) to run them locally.

- **Temporal workflow boundary tests with time-skipping** (`tests/integration/temporal/test_execution_rescheduling.py`, `tests/integration/temporal/test_interventions_temporal.py`, `tests/integration/workflows/temporal/**`) are **not** marked `integration_ci` because they consistently exceed CI timeout thresholds under the Temporal test server. They remain valuable for local dev verification.

Note: Jules **unit** tests (`tests/unit/jules/`, `tests/unit/workflows/temporal/test_jules_activities.py`, etc.) remain in the required unit suite — only Jules *provider verification* tests are excluded from required CI.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MoonLadderStudios/MoonMind](https://github.com/MoonLadderStudios/MoonMind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
