---
trigger: always_on
description: Guidance for AI coding agents working in the Plasma monorepo. The goal of this file is to let an agent contribute a complete, review-ready pull request **without having to ask a human how**. Read it before making changes.
---

# AGENTS.md

Guidance for AI coding agents working in the Plasma monorepo. The goal of this file is to let an agent contribute a complete, review-ready pull request **without having to ask a human how**. Read it before making changes.

Humans should read [CONTRIBUTING.md](CONTRIBUTING.md) and the [README](README.md) instead — they cover the same ground with more narrative.

## What Plasma is

Plasma is Coveo's design system, used in the Coveo Cloud Administration Console. It ships a Mantine-themed React component library, design tokens, React icons, and documentation. It is a monorepo managed with **pnpm workspaces** and **Turbo**.

**Import invariant:** always import components from `@coveord/plasma-mantine`, even when Mantine's own docs were the reference source.

## Repository layout

Packages live in `packages/*` (declared in `pnpm-workspace.yaml`):

| Package                              | Purpose                                                                      |
| ------------------------------------ | ---------------------------------------------------------------------------- |
| `@coveord/plasma-mantine`            | Plasma-flavoured Mantine theme and custom components (main library)          |
| `@coveord/plasma-tokens`             | Design tokens (colors, spacing, typography, …)                               |
| `@coveord/plasma-react-icons`        | React icon components (`src/generated` is generated — do not edit by hand)   |
| `@coveord/plasma-storybook`          | Storybook component documentation site                                       |
| `@coveord/plasma-figma-code-connect` | Figma Code Connect instances                                                 |
| `@coveord/plasma-llms`               | LLM-friendly component specs (`llms.txt`, `llms-full.txt`, per-component)    |
| `@coveord/plasma-mcp-server`         | MCP server exposing Plasma docs to AI agents                                 |
| `@coveord/plasma-style` / `-react`   | **Maintenance mode** — do not add features. Legacy lives on the `v53` branch |

## Tech stack

- **Node.js 24** and **pnpm** (pinned via `packageManager`; enable with `corepack enable`). pnpm is enforced — npm/yarn are blocked by `only-allow`.
- **React 19** with the **Mantine** UI library.
- **TypeScript** (strict).
- **Vitest** + **React Testing Library** for tests.
- **oxfmt** for formatting and **oxlint** for linting (not Prettier/ESLint). **Stylelint** for SCSS/CSS.
- **Turbo** for build/test orchestration, **Changesets** for versioning, **Husky** + **lint-staged** for pre-commit hooks.

## Setup

Run everything from the repository root.

```bash
pnpm install   # install and link all workspace packages
```

## Everyday commands

| Command          | What it does                                               |
| ---------------- | ---------------------------------------------------------- |
| `pnpm build`     | Build all packages (`turbo run build`)                     |
| `pnpm start`     | Build deps then start the demo/dev servers with hot reload |
| `pnpm test`      | Run all tests (`turbo run test`, runs in UTC)              |
| `pnpm lint`      | Lint with oxlint                                           |
| `pnpm fmt`       | Format with oxfmt                                          |
| `pnpm fmt:check` | Verify formatting without writing (this is what CI runs)   |
| `pnpm lintfix`   | Auto-fix lint/style issues across packages                 |
| `pnpm changeset` | Create a changeset for a releasable change                 |

You can run tests for a single package from `packages/{name}` with `pnpm test`, `pnpm test:watch`, or `pnpm test:debug`.

If `pnpm lint` fails with `Error: Invalid tsconfig`, set `OXLINT_TSGOLINT_DANGEROUSLY_SUPPRESS_PROGRAM_DIAGNOSTICS=true` (CI does this).

## Code style and conventions

Formatting is enforced by **oxfmt** (see `.oxfmtrc.json`): print width 120, tab width 4, single quotes, no bracket spacing in objects. Run `pnpm fmt` before committing — don't hand-format.

- **TypeScript** for all new code; prefer type inference; export types for public APIs.
- **React**: functional components with hooks, React 19 patterns, JSX runtime (no `import React`), prefer **named exports**.
- **File naming**: components `PascalCase.tsx`; utilities `camelCase.ts`; tests co-located as `*.spec.ts(x)`; styles as `*.module.css` / `*.css`.
- Match existing patterns in the surrounding package before introducing new ones.

## Testing

- Use **Vitest** + **React Testing Library**; test user-facing behavior, not implementation details.
- Co-locate `*.spec.ts(x)` files with the source.
- Do **not** prefix test names with "should"; use present tense: `it('returns true when value is valid')`, not `it('should return …')`.
- Cover new components/utilities and their edge cases, including accessibility.
- Add or update tests for every behavior change and confirm `pnpm test` passes before opening a PR.

## Internal skills

This repo ships agent **skills** in `.github/skills/`. Use them when the task matches — they encode the exact expected format and workflow:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coveo/plasma](https://github.com/coveo/plasma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
