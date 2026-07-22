---
trigger: always_on
description: provides `libmpv1`, so use the x64 AppImage on Jammy instead of weakening the
---

# AGENTS.md

This file provides guidance to coding agents working in this repository.

## Plan Mode

- When an agent is in Plan Mode and produces a final `<proposed_plan>`, it must also save that finalized plan as a Markdown file in the repo-root `.plans/` directory.
- Save only finalized plans. Do not write interim exploration, questions, or draft revisions to `.plans/`.
- Use the filename pattern `YYYY-MM-DD-short-topic.md` such as `.plans/2026-03-12-channel-filtering.md`.
- If the intended filename already exists, append a numeric suffix such as `-2`, `-3`, and so on.

## Agent Bootstrap

- In a fresh worktree, run `pnpm install --frozen-lockfile` before relying on Nx project discovery, lint, test, or build commands. Without `node_modules`, `pnpm nx show projects` will fail because the local Nx modules are unavailable.
- After dependencies are installed, verify workspace discovery with `pnpm nx show projects`.
- Use scoped path aliases from `tsconfig.base.json` such as `@iptvnator/services`, `@iptvnator/shared/interfaces`, and `@iptvnator/ui/components`. Do not add new imports from legacy bare aliases such as `services`, `shared-interfaces`, `components`, `m3u-state`, or `database`.
- Every Nx project should keep `scope:*`, `domain:*`, and `type:*` tags in `project.json` so `@nx/enforce-module-boundaries` remains useful for humans and agents.
- See `docs/architecture/nx-workspace-boundaries.md` for the current Nx tag and alias policy.
- Repository-specific skills are committed under `.codex/skills/`. If an external agent does not support skills, treat those files as concise ownership docs.

## Documentation After Changes

- After implementing a meaningful change, agents must assess whether canonical repo docs need updates before considering the task complete.
- Meaningful changes include new or changed user-visible behavior, architecture or data-flow changes, non-obvious maintenance workflows, new setup/debugging steps, and new subsystem contracts or boundaries.
- Skip doc updates for trivial refactors with unchanged behavior, formatting-only edits, and isolated test-only changes.
- Prefer updating an existing authoritative doc before creating a new one:
    1. `README.md` for top-level developer or user workflows
    2. `docs/architecture/` for architecture, ownership, and behavior contracts
    3. the nearest module `README.md` for local usage or behavior
- Keep the root `CLAUDE.md` and this file up to date. They are living documents: whenever a change touches something they describe — monorepo structure (new/moved/renamed apps or libs), routes, database schema/tables, stores and their features, key components, commands, environment behavior, or coding conventions — update the affected sections as part of the same task, and keep the process sections mirrored between `AGENTS.md` and `CLAUDE.md` in sync.
- When adding a new feature area, check whether the Architecture or Key Features sections of `CLAUDE.md` describe the surrounding area; if they do, reflect the addition there instead of leaving the description stale.
- Do not let `CLAUDE.md` or `AGENTS.md` drift: a stale path or route in these files poisons the context of every future agent session. If you notice an outdated claim while working, fix it (or flag it in the final summary) even if it is unrelated to the current task.
- Repo docs are canonical even when they were originally drafted by an LLM.
- Final task summaries should state whether docs were updated and which doc changed.

## Regression Prevention And Test Updates

- Before the final summary for any feature, behavior change, bug fix, data-flow change, Electron IPC/database change, or user-visible UI workflow change, complete a test impact pass. Identify the affected projects and decide whether unit, integration, E2E, build, lint, or manual/CDP verification is required.
- Bug fixes must normally include regression coverage that fails on the old behavior and passes with the fix. If automated coverage is not practical, document why in the final summary and include the strongest manual validation performed.
- Feature work and behavior changes must update existing tests when assertions, fixtures, mocks, routes, or E2E flows are now stale, incomplete, or missing. Prefer extending the closest existing spec or E2E file before adding a new suite.
- Default validation ladder:
    1. Run targeted unit tests for directly affected projects with `pnpm nx test <project>` or existing scripts such as `pnpm run test:frontend`, `pnpm run test:backend`, or `pnpm run test:unit:ci` when the scope is broader.
    2. Run affected E2E coverage when changing user-visible workflows, routing, persistence, playback, portals, settings, import flows, or Electron-only behavior.
    3. Use `pnpm nx show projects --withTarget test` and `pnpm nx show projects --withTarget e2e` when project ownership or available validation targets are unclear.
    4. Prefer specific atomized E2E targets before broad suites when they cover the changed behavior, for example `pnpm nx run web-e2e:e2e-ci--src/xtream.e2e.ts` or `pnpm nx run electron-backend-e2e:e2e-ci--src/search.e2e.ts`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [4gray/iptvnator](https://github.com/4gray/iptvnator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
