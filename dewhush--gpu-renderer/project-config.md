---
trigger: always_on
description: - This guidance is intended for Codex usage under `~/.codex/` and applies as the default operating policy unless a more specific repo-local `AGENTS.md` overrides or refines it.
---

# AGENTS.md

## 1. Scope and Purpose
- This guidance is intended for Codex usage under `~/.codex/` and applies as the default operating policy unless a more specific repo-local `AGENTS.md` overrides or refines it.
- Treat this as the current curated rule set, not as an append-only history log.
- The Plasius Ltd monorepo contains the public site, admin dashboard, backend API, and shared packages.

## 2. Instruction Priority and Conflict Handling
- Apply the most specific applicable rule.
- Repo-local `AGENTS.md` files may refine this file for a specific repository or subtree.
- Referenced companion markdown files expand this file and are part of the active instruction set.
- If rules appear contradictory and the conflict cannot be resolved safely from the written guidance, stop and ask the user how to resolve it before proceeding.
- Canonical architecture decision path: `docs/adrs/`. Any legacy references such as `docs/ADRS/`, `docs/ADR`, or similar should be merged into or interpreted as `docs/adrs/`.

## 3. Repository Map and Common Paths
- `packages/`: shared libraries (schema, auth, renderer, etc.)
- `frontend/`: public site (Vite)
- `dashboard/`: admin dashboard (Vite)
- `backend/`: Azure Functions API
- `docs/` and `specs/`: documentation and TypeSpec/OpenAPI
- Prefer editing source in `packages/`, `frontend/`, `dashboard/`, and `backend/`.
- Avoid editing generated output (`dist/`, `coverage/`, `tsp-output/`, `node_modules/`) unless explicitly requested.

## 4. Tooling and Common Commands
- Use npm (workspaces + Turbo).
- Install dependencies from the repo root with `npm install`.
- Common root commands:
  - `npm run build`
  - `npm run dev`
  - `npm run test`
  - `npm run typecheck`
  - `npm run generate:references`

## 5. Non-Negotiable Safety and Integrity Rules
- Secrets and PII must never be committed to version control.
- Sensitive values are only permitted in approved local metadata/config (for example `.env*`) and managed secret stores.
- If exposure occurs, treat it as a blocking incident and rotate/remediate immediately.
- Do not fake completion, CI status, CD status, release status, project status, or test execution.
- Do not publish packages directly from local machines; publish only through approved GitHub CD workflows.
- Do not bypass approved production deployment paths.
- Do not use ad hoc production deployment via manual Azure CLI changes, alternate workflows, or undocumented scripts.
- Keep public APIs stable unless the work intentionally changes them.

## 6. Core Engineering Rules
- Reuse `@plasius/*` packages as the default building blocks for all new features and tasks.
- Before implementing new functionality, evaluate existing `@plasius/*` packages and reuse them where they fit.
- Do not reimplement capabilities that already exist in `@plasius/*` packages.
- If needed behavior is missing, update the appropriate `@plasius/*` package first, including tests and docs, before consuming the new released version.
- Prefer durable fixes over short-term bypasses.
- Quick fixes are not acceptable for test coverage, CI/CD pipelines, or documentation updates.
- Do not remove tests, checks, or docs, lower quality thresholds, or bypass required cases merely to make work pass.
- Apply SOLID, KISS, and related engineering principles where appropriate.
- Preserve ACID properties and data integrity in transactional workflows where applicable.
- Keep code clean, maintainable, scalable, and testable.
- When adding or updating dependencies, prefer lazy-loading (dynamic import/code splitting) to avoid heavy first-load network use when applicable.
- Accessibility is a first-class quality requirement. User-facing software should target WCAG 2.2 AA or better, and accessibility regressions are release-blocking.

## 7. Work Definition and GitHub Project Governance
- GitHub Project is the source of truth for work definition, acceptance criteria, ownership, and completion state.
- Do not use repo comments, ad hoc conversations, or untracked local notes as the authoritative work definition for implementation.
- Design documents may precede tracked work, but implementation work should be represented in the GitHub Project hierarchy before execution.
- Work hierarchy is `Epic -> Feature -> Story -> Task`.
- Work items are GitHub Issues titled with prefixes:
  - `[EPIC]`
  - `[FEATURE]`
  - `[STORY]`
  - `[TASK]`
- All `Epic`, `Feature`, and `Story` items must be created and managed in the `plasius-ltd-site` repository.
- All `Task` items must be created in the repository/package where the code change will be implemented.
- When one Story requires changes in multiple repositories/packages, create one linked Task per affected repository/package.
- Every implementation Task must belong to a parent Feature. Do not create or execute standalone implementation Tasks outside a Feature.
- Non-implementation maintenance tasks should still be linked to a parent Feature when they are part of delivery scope. If they are truly repository-operational work, document explicitly why no product Feature applies.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dewhush/gpu-renderer](https://github.com/dewhush/gpu-renderer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
