---
trigger: always_on
description: This file tells AI coding agents (Claude, Cursor, Copilot, etc.) how to work in this repository.
---

# AGENTS.md — Canvas Kit

This file tells AI coding agents (Claude, Cursor, Copilot, etc.) how to work in this repository.
It is the entry point — read it before making changes. Deeper styling/token rules live in
[STYLE.md](./STYLE.md); link out to it rather than duplicating it.

## What this repo is

Canvas Kit is Workday's design-system component library, consumed by many downstream
applications across the company. It ships as versioned npm packages
(`@workday/canvas-kit-react`, `-labs-react`, `-preview-react`, `-styling`, `-css`, ...).

This changes how you should behave compared to an application repo:

- **Every public API change affects downstream consumers.** Prop renames, removed exports,
  changed defaults, and behavior changes ripple into consumer apps that may be several major
  versions behind. Breaking API changes belong on `prerelease/major`; additive, non-breaking
  features belong on `prerelease/minor`.
- **Nothing is "just internal cleanup."** Build config, `tsconfig*.json`, lint rules, and package
  entry points affect what downstream bundlers and type-checkers see. Treat these as public
  surface area, not implementation detail.
- **Design tokens are the styling contract.** Visual changes should come from
  `@workday/canvas-tokens-web`, not ad hoc values. See [STYLE.md](./STYLE.md).

When in doubt about impact, say so explicitly and ask rather than guessing.

## Branching strategy

Canvas Kit maintains three major versions at once, across four long-lived branches (full detail:
[MAINTAINING.mdx § Branches](modules/docs/mdx/MAINTAINING.mdx)):

| Branch | Version | What's allowed there |
|---|---|---|
| `support` | previous major | Patches only — no new features, no breaking changes |
| `master` | current major | Patches and small updates. Visual/styling changes that align components to a new design direction (e.g. the v16 Sana visual update) can land here **without** an API break. Breaking API changes do not. |
| `prerelease/minor` | current major | New features (non-breaking) for the current major |
| `prerelease/major` | next major | New features, patches, **and** breaking API changes |

Changes forward-merge one direction only: `support` → `master` → `prerelease/minor` →
`prerelease/major`. A fix made in `support` eventually reaches every later branch; a feature or
breaking change made in `prerelease/major` never automatically flows backward.

What this means when you're deciding where a change belongs:

- **Visual-only changes** (colors, spacing, shape that follow updated tokens, with no prop/API
  change) can target `master`.
- **New, additive, non-breaking features** target `prerelease/minor`, not `master`.
- **Breaking changes** — removed/renamed exports, changed prop defaults, altered behavior — belong
  on `prerelease/major` only. If you're not on that branch and a change looks breaking, say so and
  ask before proceeding; don't assume `master` can absorb it because "it's just this once."
- If you're unsure which branch a PR should target, ask — don't guess based on branch name alone,
  and don't assume `master` is the conservative, breaking-change-free branch in every dimension
  (it isn't, for visual changes) or that it's safe for API changes (it isn't).

## Before you touch config

Do **not** modify `tsconfig.json`, `tsconfig.*.json`, `styling.config.ts`, `vite.config.ts`,
`vitest.config.ts`, `babel.config.js`, `eslint.config.js`, or anything under `.github/workflows/`
unless the task explicitly calls for it. These files are shared across every package in the
monorepo and shape what gets published.

If a task seems to require a config change:

1. State which file and which setting, and explain *why* the change is needed and what it
   affects (build output, type-checking behavior, downstream consumers, CI).
2. Prefer the smallest, most targeted change (a single compiler option, a single override block)
   over a broad rewrite.
3. Call out the blast radius (e.g., "this changes `target` for every published package") before
   making the edit, even if you're confident it's correct.

Never make a speculative or "might as well" config change alongside an unrelated fix.

## AI tooling artifacts — do not commit

Cursor and Claude (and similar agents) write local working files that are **not** part of this
repo's source. Never add, stage, or commit them:

- **Cursor** — plan files (e.g. `.cursor/plans/`), debug sessions, and other session state under
  `.cursor/`
- **Claude** — plan files, debug sessions, and other local session artifacts (e.g. under
  `.claude/`)

These are personal/editor-local scratch work. If you create them while working on a task, leave
them out of the PR entirely — don't check them in "for reference" or tuck them into an unrelated
commit. If one appears in `git status`, do not stage it.

## Tooling & environment

- **Package manager:** Yarn (classic), workspaces under `modules/**`. Do not use `npm install`.
- **Node:** Use Node 24 — [.nvmrc](./.nvmrc) pins `24.16.0`, and CI installs Node `24.x`
  (`.github/workflows/pull-request.yml`).
- **Monorepo tool:** Lerna, driving `build`, `clean`, `depcheck`, `watch` across `modules/**`.
- **TypeScript:** 5.0, `strict: true`. Don't relax strictness to make an error disappear.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Workday/canvas-kit](https://github.com/Workday/canvas-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
