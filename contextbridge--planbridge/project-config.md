---
trigger: always_on
description: A Bun-based CLI that brings human-in-the-loop annotation into AI coding sessions. The first capability is **plan review**: a user (or a harness hook) feeds a proposed plan into `contextbridge plan`, the CLI opens a local browser UI where a human annotates or approves, and the result is returned on stdout so the caller — typically a Claude Code or Codex hook — can iterate.
---

# AGENTS.md

## Project: ContextBridge CLI (`contextbridge`)

A Bun-based CLI that brings human-in-the-loop annotation into AI coding sessions. The first capability is **plan review**: a user (or a harness hook) feeds a proposed plan into `contextbridge plan`, the CLI opens a local browser UI where a human annotates or approves, and the result is returned on stdout so the caller — typically a Claude Code or Codex hook — can iterate.

The longer arc is a multi-tool CLI. Plan review is feature one; code review (`contextbridge review`), additional annotation surfaces, and session-data mining live as sibling subcommands on the same binary.

## Stack

- **Runtime:** Bun
- **Language:** TypeScript (strict)
- **Monorepo:** bun workspaces with a flat `packages/` tree. No turbo.

## Repository layout

```
planbridge/
├── packages/                  # public bun workspace ("workspaces": ["packages/*"])
│   ├── cli/                   # @contextbridge/cli — the `contextbridge` binary
│   ├── context/               # @contextbridge/context — shared BaseContext + FrontendContext
│   ├── instrumentation/       # @contextbridge/instrumentation — PostHog + Sentry wrappers (node + browser)
│   ├── shared/                # @contextbridge/shared — types + zod schemas shared across packages
│   ├── server/                # @contextbridge/server — local Bun.serve HTTP library
│   ├── ui/                    # @contextbridge/ui — shared CSS, fonts, cn(), shadcn components
│   └── plan/                  # @contextbridge/plan — Vite+React browser UI for plan review
├── tools/                     # private git submodule for employees/trusted CI only
│   ├── projen/                # projen runner (uses private @contextbridge/projen)
│   ├── infrastructure/        # AWS CDK stacks + release pipeline (uses private @contextbridge/infrastructure)
│   └── website/               # @contextbridge/website — Astro + Starlight marketing/docs site (projen-managed, ships as a CDK asset)
├── tsconfig.base.json         # shared TS compiler options
├── package.json               # root workspace ("workspaces": ["packages/*"])
└── justfile                   # root-level recipes
```

`tools/` is a private git submodule backed by `contextbridge/planbridge-private`, not public source in this repository. Public contributors should not initialize it; a normal clone plus root `bun install` and `just verify` are sufficient. Employees and trusted CI with access can run:

```sh
just install
just tools-init
```

Tools changes are developed in the private repo and then pinned here by updating the submodule pointer. Projen synth may modify both repositories; commit private `tools/` changes in `planbridge-private` and public generated changes in `planbridge` separately.

Package naming: every workspace is `@contextbridge/<short-name>`. Review-experience UIs are named by feature (`plan`, later `review`) — never a generic `-ui` suffix. Future review surfaces (`packages/review` for file-change review, etc.) land as siblings. Libraries that multiple experiences share (shared contracts, context, server) are their own packages.

Each package has its own `AGENTS.md` with package-specific guidance (plus a one-line `CLAUDE.md` stub that imports it via `@AGENTS.md` so Claude Code auto-loads it when editing files in that directory). **The stub is load-bearing** — Claude Code discovers ancestor `CLAUDE.md` on file edits but not standalone `AGENTS.md`; don't drop it. `packages/shared` and `packages/server` don't have their own files — they have no package-specific guidance beyond root conventions.

### `tools/website` (projen-managed)

The marketing/docs site lives in the private `planbridge-private` submodule rather than `packages/` because it deploys as a CDK asset of `MarketingWebsiteStack` — keeping source colocated with the stack that ships it. It is the only projen-managed package in the monorepo. Projen config lives in the same submodule at `tools/projen/.projenrc.ts`: a bare `projen.Project` acts as a shell that hosts a `GitHub` component plus the website as a child subproject (`outdir: '../..'` from the submodule). Public contributors do not need to initialize the submodule or run projen synth; building the website locally requires `just tools-init` first.

## Verification

Before marking a task complete, run `just verify` and fix anything that fails. It runs four steps in order:

- `bun run format:check` — Prettier
- `bun run typecheck` — strict TypeScript check (`bun run --filter '*' typecheck`)
- `bun run lint` — ESLint (`--max-warnings 0`)
- `bun run test` — dispatches per-package `test` scripts. Most packages use Bun's test runner; `@contextbridge/plan` uses **vitest** (browser mode via Playwright/Chromium) because the plan UI tests depend on real DOM, CSS Custom Highlights, and selection APIs that Bun's runner can't provide.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [contextbridge/planbridge](https://github.com/contextbridge/planbridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
