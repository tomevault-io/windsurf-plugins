---
trigger: always_on
description: A single-process framework for building sites on Next.js with an inline content-editing UI. The Next.js application owns rendering, content storage, and admin endpoints — there is no second process and no agent channel. The stack is frozen: Next.js only. Deploy is a regular Next.js project (Vercel by default — no containers). The admin/preview UI is a dev-time tool; production deploys are read-only. A developer's local Claude Code can optionally load skills from `.claude/skills/` to scaffold sec
---

# agntcms

A single-process framework for building sites on Next.js with an inline content-editing UI. The Next.js application owns rendering, content storage, and admin endpoints — there is no second process and no agent channel. The stack is frozen: Next.js only. Deploy is a regular Next.js project (Vercel by default — no containers). The admin/preview UI is a dev-time tool; production deploys are read-only. A developer's local Claude Code can optionally load skills from `.claude/skills/` to scaffold sections and manage pages, but the framework does not require it. See `@ARCHITECTURE.md`.

Stage: v0.5 — channel/agent infrastructure removed in lockstep. Framework not yet released. No users, breaking changes are fine until the v1.0.0 tag.

## Repository layout

```
packages/
├── next/        # @agntcms/next         runtime, React, route handlers
├── skills/      # @agntcms/skills       Claude Code skills, source of truth for project structure
└── cli/         # create-agntcms-app    (unscoped) — bundles template at build time
template/        # agntcms-template      reference Next.js project
```

All four units are released in lockstep under one version. Dependency graph: `cli → skills`, `template → next, skills`. `next` and `skills` are isolated.

## Commands

- `pnpm dev` parallel watch across all packages
- `pnpm test` vitest across all packages
- `pnpm test packages/next/src/domain` targeted run
- `pnpm typecheck` tsc across all packages, must be green before any commit
- `pnpm build` production build of all packages
- `pnpm template:dev` boot the template as a live sandbox against locally linked packages

## Hard invariants

These rules cannot be broken without an explicit decision from the lead agent and a corresponding update to `@ARCHITECTURE.md`.

1. Dependencies inside `@agntcms/next` flow strictly one way: `domain ← storage ← runtime ← handlers`. `react` depends only on `domain`, on type imports from `runtime`, and on `sections` (types and the slot-wrapping helpers `wrapSectionProps` / `wrapAsSlot` — see ARCHITECTURE.md §7). No cycles.
2. `react` imports nothing from `storage`. Server dependencies must not leak into the client bundle.
3. The subpath exports `@agntcms/next`, `/server`, `/client`, `/handlers`, `/config` are public contract. Change only deliberately, with a breaking-change note.
4. The template directory layout is the public contract of the framework. Skills are its single source of truth. Any change to folder structure or frozen files is a breaking change and requires synchronized updates to `skills` and `template`.
5. The frozen zone in the template is exactly `app/api/agntcms/`, `app/[[...slug]]/`, `app/not-found.tsx`, `.claude/settings.json`, and `.claude/skills/` — not user-editable. `app/sitemap.ts` and `app/robots.ts` ship as working defaults but are user-editable, not frozen. `.claude/launch.json` is a Claude Code harness file (not framework-owned, gitignored), also not frozen. The admin surface is served by a single catch-all `app/api/agntcms/[...path]/route.dev.ts` (the `.dev.ts` suffix removes it from the production build); the entire admin surface disappears in `next build`. Any feature that requires touching the frozen zone is either a design mistake or a breaking framework change.
6. No codegen, no folder scanning. Section registration is always explicit through `agntcms/config.ts`.
7. KISS and YAGNI. If a feature is not in `@ARCHITECTURE.md` sections 1 through 10, by default it does not exist. The deferred list lives in section 11.

## Working discipline

- **Root cause before fix.** Before editing code to fix a bug, state one explicit hypothesis about the cause and verify it (read the relevant code, check a log, run a focused test). If the hypothesis is not confirmed, do not try another edit "just in case" — form a new hypothesis and verify again. Speculative tweaking is how three wrong fixes land before the real one.
- **No-op verify stops.** On "compare / sync / check that X matches Y" tasks: diff first. If there is no divergence, stop and report "aligned, no changes needed" in one line. Do not expand scope into neighboring files or propose refactors.
- **Stay in the requested directory.** If the user points at a specific path (`temp/`, `packages/next/src/runtime/`, a single file), do not wander up the tree or touch sibling packages without explicit permission.

## Code style

- TypeScript strict, no `any`. `unknown` with explicit narrowing is fine.
- No default exports in library code, named only.
- Domain types live in `packages/next/src/domain` and are not redefined anywhere else.
- Tests sit next to code: `foo.ts` plus `foo.test.ts`. Unit tests for `domain` are mandatory, for `runtime` desirable, for `handlers` covered through integration tests.
- Comments explain **why**, not **what**. Obvious code stays uncommented.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agntcms/agntcms](https://github.com/agntcms/agntcms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
