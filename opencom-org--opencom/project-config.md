---
trigger: always_on
description: - Use **PNPM** commands in this repo (workspace uses `pnpm-workspace.yaml`).
---

# AGENTS

## Non-negotiables

- Use **PNPM** commands in this repo (workspace uses `pnpm-workspace.yaml`).
- Always run new/updated tests after creating or changing them.
- Prefer focused verification first (targeted package/spec), then broader checks when needed.
- At the end of each proposal when ready for a PR, run `pnpm ci:check` to ensure all checks pass.

## Quick Repo Orientation

- Monorepo root: `opencom`
- Main apps: `apps/web`, `apps/landing`, `apps/mobile`, `apps/widget`
- Backend: `packages/convex`
- OpenSpec source of truth: `openspec/changes/<change-name>/`

## General Workflow Guardrails

- Start every non-trivial task by grounding in current repo state before changing files:
  1. identify the active scope
  2. read the relevant files/specs/tests
  3. verify whether the work is already partly done
  4. choose a narrow verification plan
- If working from an existing OpenSpec change, always read:
  - `openspec status --change "<change-name>" --json`
  - `openspec instructions apply --change "<change-name>" --json`
  - the current `proposal.md`, `design.md`, `specs/**/*.md`, and `tasks.md`
- Never assume unchecked boxes in `tasks.md` mean the code is still missing. Verify the current implementation first, then update artifacts or tasks to match reality.
- Before creating a new OpenSpec change, quickly check for overlapping active changes or existing specs so you do not create duplicates or split ownership accidentally.
- For multi-step work, keep an explicit plan/todo and update it as tasks complete. Prefer one active task at a time.
- When changing course mid-task, record the new scope and the reason in the active change artifacts if they are affected.
- Before marking work complete, verify both code and artifacts:
  - code/tests/typechecks reflect the final state
  - `tasks.md` checkboxes match what is actually done
  - any follow-up work is written down explicitly instead of left implicit

## Existing Proposal Discipline

- If you did not create the current proposal/change, treat the artifacts as hypotheses until verified against the codebase.
- Separate findings into three buckets before editing artifacts:
  - already implemented
  - still unfinished
  - intentionally out of scope or accepted exception
- Only put unfinished work into active proposal/spec/task artifacts.
- If code and artifacts disagree, prefer fixing the artifact first unless the user explicitly asked for implementation.
- When leaving partial progress, record exact remaining file clusters, blockers, and verification still needed so a later pass can continue without re-auditing the whole repo.

## High-Value Commands (copy/paste)

### Typecheck

- Convex only:
  - `pnpm --filter @opencom/convex typecheck`
- Web only:
  - `pnpm --filter @opencom/web typecheck`
- Whole workspace:
  - `pnpm typecheck`

### Convex TypeScript deep-instantiation workaround

- Canonical guide: `docs/convex-type-safety-playbook.md`
- If Convex typecheck hits `TS2589` (`Type instantiation is excessively deep and possibly infinite`) at generated refs like `api.foo.bar` or `internal.foo.bar`, prefer a **local escape hatch** instead of broad weakening.
- First keep call signatures shallow at the hot spot:
  - cast `ctx.scheduler.runAfter`, `ctx.runQuery`, or `ctx.runMutation` to a local shallow function type.
- If merely referencing `api...` / `internal...` still triggers `TS2589`, use `makeFunctionReference("module:function")` from `convex/server` at that call site instead of property access on generated refs.
- Keep this workaround **localized only to pathological sites**. Continue using generated `api` / `internal` refs normally elsewhere.
- Expect hidden follow-on errors: rerun `pnpm --filter @opencom/convex typecheck` after each small batch of fixes, because resolving one deep-instantiation site can reveal additional ones.

## Convex Type Safety Standards

- Read `docs/convex-type-safety-playbook.md` before adding new Convex boundaries.
- Frontend runtime/UI modules must not import `convex/react` directly. Use local adapters and wrapper hooks instead.
- Keep Convex refs at module scope. Never create `makeFunctionReference(...)` values inside React components or hooks.
- Do not add new `getQueryRef(name: string)`, `getMutationRef(name: string)`, or `getActionRef(name: string)` factories.
- Backend cross-function calls should use generated `api` / `internal` refs by default. Only move to fixed `makeFunctionReference("module:function")` refs after a real `TS2589` hotspot is confirmed.
- Keep unavoidable casts localized to adapters or named backend hotspot helpers. Do not spread `as unknown as`, `unsafeApi`, or `unsafeInternal` through runtime code.
- After changing a boundary, update the relevant hardening guard:
  - `packages/convex/tests/runtimeTypeHardeningGuard.test.ts`
  - `apps/web/src/app/typeHardeningGuard.test.ts`
  - `apps/widget/src/test/refHardeningGuard.test.ts`
  - `packages/react-native-sdk/tests/hookBoundaryGuard.test.ts`

## Convex Hardening Audit Triage

- Before treating an audit item as open work, verify whether it is already implemented and only the guard/proposal text is stale.
- Default classification for current repo state:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opencom-org/opencom](https://github.com/opencom-org/opencom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
