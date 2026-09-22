---
trigger: always_on
description: This repository is designed to be developed incrementally by AI coding agents.
---

# AGENTS.md — Operating manual for AI agents (and humans)

This repository is designed to be developed incrementally by AI coding agents.
**Read this file before touching any code.** It defines the hard rules, the workflow, and the
definition of done. The product spec lives in [docs/BLUEPRINT.md](docs/BLUEPRINT.md); the backlog
in [docs/TASKS.md](docs/TASKS.md); the current position in [docs/STATE.md](docs/STATE.md).

---

## 1. Hard rules (non-negotiable)

1. **Backend is JavaScript only.** Node.js, ESM (`type: module`). **TypeScript is forbidden** —
   no `.ts` files, no `tsc`, no type-only tooling. CI enforces this (`npm run check:no-ts`).
   Use JSDoc comments for editor type hints where helpful.
2. **Database is MySQL** (8.x), accessed through **knex** + **mysql2**. No ORMs, no other databases
   for canonical data. Redis is for queues/cache/realtime fanout only.
3. **All client platforms are one Flutter codebase** (`apps/app`). No secondary web framework
   unless a task explicitly justifies it with an ADR.
4. **Every feature ships with tests.** API: Vitest (unit + integration). App: `flutter test`.
   A task is not done if `npm test` or `flutter test` fails.
5. **Every task updates the docs.** At minimum: check the box in `docs/TASKS.md`, update
   `docs/STATE.md`, add a `CHANGELOG.md` entry. Update `docs/ARCHITECTURE.md` when structure changes.
6. **Architectural decisions require an ADR** in `docs/adr/` (use the template). Examples: new
   dependency category, schema redesign, protocol change, security-relevant choice.
7. **Never commit secrets.** `.env` is gitignored; only `.env.example` is committed, with
   placeholder values. OAuth tokens are stored encrypted (see BLUEPRINT §15.3).
8. **Migrations are append-only.** Never edit an applied migration; create a new one.
   Naming: `YYYYMMDDHHMMSS_verb_subject.js` with ESM `export async function up/down(knex)`.
9. **Conventional Commits.** `feat(api): …`, `fix(app): …`, `docs: …`, `chore: …`, `ci: …`,
   `refactor(api): …`, `test(api): …`. Scope is `api`, `app`, or omitted for repo-wide.
10. **Do risky things in writing first.** Large refactors and data migrations get a short plan
    (in the task section of `docs/TASKS.md` or an ADR) *before* implementation.
11. **One design system, forever.** All UI follows the "AllisWell Glass" design language defined
    in [docs/DESIGN.md](docs/DESIGN.md) (ADR-0005) — **visual continuity is mandatory for every
    future feature**, screen and platform. Concretely: colors/spacing/radii come from
    `apps/app/lib/src/theme/` tokens (no raw hex or `Colors.*` in widgets), glass/blur is
    chrome-only (never under body text), text contrast ≥ 4.5:1 and icon/border contrast ≥ 3:1 in
    BOTH themes (`python3 scripts/design/contrast.py` must pass after palette edits), tap targets
    ≥ 44 px, and every UI change is checked in light *and* dark before it is done. Deviations
    require amending docs/DESIGN.md in the same change.
12. **The MCP tool surface and the public API are part of every feature.** When a task adds or
    changes a user-facing capability (an entity, a field, an operation), the same epic must
    extend the remote MCP server (and [docs/MCP.md](docs/MCP.md)) and the key-authenticated
    public REST surface ([docs/API.md](docs/API.md)) to match — or record a one-line written
    reason in the task. Standing exceptions, already decided: `delete_*` never enters MCP
    (ADR-0022; the API-key surface does expose deletes), and raw file bytes / presigned URLs
    never flow through MCP (AI.md §7). Added in the Epic 25 planning round (2026-08-13);
    OPH-261…OPH-266 backfill the gaps that existed on that date.

## 2. The "do the next task" protocol

When the user says **“do the next task”** / **“sıradaki işi yap”** (or similar):

1. **Locate position.** Read `docs/STATE.md` → “Next task”. Cross-check `docs/TASKS.md`
   (first unchecked `[ ]` task in the current epic; epics are ordered).
2. **Understand scope.** Read the task's checklist, acceptance criteria and tests. Read the
   relevant BLUEPRINT sections. Look at existing code — reuse existing helpers and patterns.
3. **Implement** the task fully, following the hard rules above. Small, cohesive diffs.
4. **Verify.** Run `npm run lint`, `npm test` (and `npm run test:integration` when infra is
   running; `flutter analyze` + `flutter test` for app changes). Fix what breaks.
5. **Document.** Mark the task `[x]` in `docs/TASKS.md`, update `docs/STATE.md` (last completed,
   next task, any new notes/risks), add a `CHANGELOG.md` line, update other docs if needed.
6. **Commit** with a Conventional Commit message referencing the task id, e.g.
   `feat(api): add register endpoint (OPH-020)`.
7. **Report** briefly: what was done, how it was verified, what is next.

Never skip ahead (dependencies are encoded in epic order). If a task is blocked, record why in
`docs/STATE.md` → “Blocked / notes”, pick the next unblocked task, and tell the user.

## 3. Definition of Done (checklist)

- [ ] Code follows hard rules (JS-only backend, MySQL, ESM, tests).
- [ ] `npm run lint` + `npm run format:check` pass.
- [ ] `npm test` passes; integration tests pass if infra available (they always run in CI).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mahirozdin/alliswell](https://github.com/mahirozdin/alliswell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
