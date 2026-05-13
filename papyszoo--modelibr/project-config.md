---
trigger: always_on
description: This file is the single workflow source of truth for repository AI work.
---

# Modelibr AI Orchestrator

This file is the single workflow source of truth for repository AI work.
Keep orchestration here.
Keep implementation patterns in scoped files under `.github/instructions/`.
Keep focused delegation logic in `.github/agents/`.

## Project Baseline

Modelibr is a self-hosted game asset library for artists and game developers.
It runs locally, stores assets locally, and must not depend on remote runtime services for core product behavior.

- Backend: .NET 9.0 Web API using Clean Architecture and DDD
- Frontend: React + TypeScript
- Worker: Node.js asset processor under `src/asset-processor/`
- Blender integration: Python addon plus Blender CLI workflow
- Database: PostgreSQL
- Orchestration: Docker Compose
- Storage: hash-based deduplication

## Invariants

- Keep the product local-first. Do not add hosted AI services, external inference APIs, or CDN-only runtime dependencies.
- Keep environment configuration in the root `.env` flow unless an existing toolchain requires a dedicated build-time file such as `src/frontend/.env.demo`.
- Use PostgreSQL behavior as the baseline for application and test decisions.
- Route frontend HTTP through feature-local API modules under `src/frontend/src/features/*/api/` backed by `src/frontend/src/lib/apiBase.ts` (axios). `ApiClient.ts` is a re-export facade for backward compatibility — do not add new fetch logic there.
- Use React Query for server state (queries + mutations) and Zustand stores for UI state (panels, navigation, preferences). Use `useState` only for component-local ephemeral state.
- When task intent is unclear or a major decision is required, explicitly ask for clarification in the conversation and pause implementation until the direction is confirmed.

## One Flow

The main agent stays responsible for synthesis, approval handling, and final QA.
Delegate specialized work to the scoped agents documented in `.github/agents/` only when that workstream is actually in scope.

### 1. Plan

- Invoke `plan` to restate the request, map affected layers, identify tests early, and propose a concrete change set.

### 2. Audit The Plan

- Invoke `plan-audit` to compare the proposed plan against existing code and docs.
- Use it to find holes, naming conflicts, missed files, missing tests, or documentation drift before editing starts.

### 3. Decide Workstreams

Dispatch only the agents required by the touched areas.
Do not pull backend guidance into a frontend-only task, and do not pull demo or docs reviewers unless the change can actually affect them.

| Workstream        | Use When                                                                                                            |
| ----------------- | ------------------------------------------------------------------------------------------------------------------- |
| `backend`         | `src/WebApi/**`, `src/Application/**`, `src/Domain/**`, `src/Infrastructure/**`, `src/SharedKernel/**` are changing |
| `frontend`        | `src/frontend/**` is changing                                                                                       |
| `asset-processor` | `src/asset-processor/**` is changing                                                                                |
| `e2e`             | `tests/e2e/**` is changing or UI behavior needs E2E coverage                                                        |
| `docs`            | user-facing behavior, API contracts, worker behavior, testing guidance, README, feature docs, or docs-video relevance may have changed |
| `demo`            | frontend-visible behavior, demo mocks, demo assets, or `build:demo` output may be affected                          |

### 4. Implement

- Delegate implementation to the minimal set of workstream agents.
- Those agents must follow only the scoped instruction files relevant to the files they edit.
- If a delegated implementation discovers cross-layer scope expansion, surface it back to the main agent instead of silently spreading further.

### 5. Check Docs

- Invoke `docs` when the change may affect `README.md`, `docs/docs/ai-documentation/*.md`, `.env.example`, or user-facing feature docs.
- Treat docs freshness as a required maintenance task, not an optional follow-up. When behavior changes, check whether `README.md` and the matching docs pages — especially `docs/docs/features/*.md` — still describe the product correctly.
- When a change touches a documented feature, have `docs` decide whether the feature video or docs-video script should change too, and whether selectors or scripted UI steps under `docs/videos/` have gone stale.

### 6. Check Demo Mode

- Invoke `demo` when the change may affect `src/frontend/.env.demo`, demo mocks, demo asset preparation scripts, or the demo build behavior.

### 7. QA

The main agent owns final verification.

- After any code change, run targeted checks while iterating and rerun the affected suites after each meaningful edit until they are green.
- Before closing a code change, run the full required local QA suites for every affected layer.
- Do not finish a session with known failing checks. Failures found while verifying a change must be investigated and fixed in the same session unless the user explicitly redirects scope.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Papyszoo/Modelibr](https://github.com/Papyszoo/Modelibr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
