---
trigger: always_on
description: SBC is a sing-box configuration builder. The product goal is a React Flow visual editor backed by a canonical sing-box JSON/domain model.
---

# SBC Agents Guide

SBC is a sing-box configuration builder. The product goal is a React Flow visual editor backed by a canonical sing-box JSON/domain model.

## Source Of Truth

Read these before changing product behavior, schema, validation, or node UI:

- [SBC React Flow R&D Plan](docs/sbc-react-flow-rd-plan.md)
- [sing-box Config Document Inventory](docs/sing-box-config-doc-inventory.md)
- [sing-box Canvas Configuration Guide](docs/sing-box-canvas-configuration-guide.md)
- [sing-box Config Capability Audit](docs/sing-box-config-capability-audit.md)
- [Goal-Driven Development](docs/goal-driven-development.md)
- `vercel-react-best-practices` skill when writing, reviewing, or refactoring React/Next.js code.

The canvas is never the config source file. `SingBoxConfig` / domain model is the source of truth; React Flow nodes and edges are a derived editing view.

## Non-Negotiables

1. **Canonical config first**: all edits go through domain commands that update canonical JSON/domain state. Do not derive final `config.json` from React Flow node data.
2. **stable-first**: default templates, export, fixtures, and blocking validation target sing-box stable. testing is explicit opt-in.
3. **Different binaries**: stable configs are checked with `sing-box-stable`; testing configs are checked with `sing-box-testing`.
4. **Document traceability**: every schema field, node type, Inspector field, and fixture must map to an entry in `docs/sing-box-config-doc-inventory.md`.
5. **Rules are ordered tables**: `route.rules` and `dns.rules` are ordered lists. The canvas may visualize references, but it must not be the ordering source.
6. **Tag references are explicit**: tag rename, delete, connect, and disconnect must update references through tested domain commands.
7. **Signed commits only**: commits must be signed and pass this repo's `pre-push` signature verification.
8. **Small atomics**: one concern per commit. Prefer changes under 400 logical lines; split larger work.
9. **No silent validation gaps**: if `sing-box check` cannot run, state that clearly in the final answer and keep schema/semantic validation separate from official validation.
10. **No unrelated cleanup**: do not refactor unrelated files while implementing a goal.
11. **React performance discipline**: frontend implementation and review must apply the `vercel-react-best-practices` skill, especially bundle size, rerender control, and async/data waterfall avoidance.

## Frontend Skill Gate

Any change that touches frontend implementation, frontend architecture, UI tests, or frontend review must use the `vercel-react-best-practices` skill in that same work session.

This is a hard gate for files such as `src/**/*.tsx`, `src/**/*.ts` used by UI state/rendering, `src/styles.css`, React Flow canvas code, component tests, Playwright UI tests, and build/bundle configuration.

Before editing frontend code:

- load/read the `vercel-react-best-practices` skill;
- identify the frontend-specific performance risks for the atomic, especially bundle size, rerender scope, expensive derived state, and async/data waterfalls;
- keep transient hover/drag/canvas interaction state out of broad canonical config subscriptions.

Before marking frontend work reviewed or done:

- explicitly review the diff against `vercel-react-best-practices`;
- verify heavy editors or optional panels are deferred where practical;
- prefer narrow Zustand/selectors and memoized derived graph data over broad rerender paths;
- record any intentional deviation in the goal doc or final milestone report.

## Development Protocol

Before editing:

- Read this file and the relevant docs above.
- Identify the active goal or task.
- Define the atomic scope: files allowed, expected behavior, tests/checks.
- Check current worktree state with `git status --short --branch`.

During implementation:

- Keep config/domain logic separate from canvas layout.
- Add or update docs when behavior, schema, or validation policy changes.
- Prefer registry-driven node/schema/form additions over ad hoc component branching.
- For React/Next code, apply `vercel-react-best-practices`: direct imports, lazy-load heavy editors, avoid broad state subscriptions, memoize expensive derived graph work, and keep frequent canvas hover/drag state out of global rerender paths.
- Preserve existing user changes; never discard unrelated work.

Before committing:

- Run `git diff --check`.
- Run available project tests/checks. If the project has no test harness yet, say so.
- For config fixtures, run the matching `sing-box-stable check` or `sing-box-testing check` once binaries are available.
- Inspect the diff for scope creep.

After committing:

- For PR work, push the branch and open the PR immediately after local checks and signed commit verification pass.
- Do not wait on unreliable GitHub Actions before opening or advancing a PR. Prefer local required checks, local E2E/smoke verification, signed commit verification, and relevant provider deployment status.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JegoVPN/sbcv](https://github.com/JegoVPN/sbcv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
